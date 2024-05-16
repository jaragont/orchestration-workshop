---
title: SQLAlchemy with asyncio
author: Aya Elsayed, Rhythm Patel
category: SQLAlchemy Workshop
date: 2024-01-07
layout: post
---

In the last step, we learned about `asyncio`.
Now let's see how we can use SQLAlchemy with an async service.
To start, checkout the branch `step-6-asyncio-sqlalchemy-base`:

```sh
git checkout step-6-asyncio-sqlalchemy-base
```

This is an async version of the service we saw in `main`.
It uses the [`aiosqlite`](https://pypi.org/project/aiosqlite/), the `async` version of `sqlite`.

Since we've added a new packages, we need to re-install our `pip` requirements:

```sh
python3.12 -m pip install -r requirements.txt
```

Now, let's read through a few differences between the sync and async versions.

### db_accessor.py

You'll notice here the implementations of the functions that execute queries has changed to use `aiosqlite`.
With `aiosqlite`, we can `await` the I/O interactions with the database so that the coroutine that is idle while waiting for the I/O interaction to complete can give a chance to other tasks in the meantime.

This means that we need to define these functions as coroutines; we achieve that by using the keyword `async def`.

#### Async Context Managers

Our execute query functions can now make use of async features like `async with`, a keyword that allows us to use asynchronous context managers.

As you may expect, `async with` is the async version of the `with` keyword.
Parallel to it's sync counterpart, using an async context manager allows you to invoke `__aenter__()` and `__aexit__()` for of your managed object.
Unlike the sync version, `__aenter__()` and `__aexit__()` are awaited.
This allows other tasks to execute while your task waits for the (potentially I/O bound) setup and cleanup of your managed object.

Let's look at `execute_query()` in our code as an example.
`__aenter__()` of `aiosqlite.connect(DB_PATH)` starts a database connection, and `__aexit__()` cleans up and closes the connection so we don't have to explicitly call `connection.close()` ourselves.

##### marketsvc/db_accessor.py

```py
async def execute_query(query, params):
    async with aiosqlite.connect(DB_PATH) as db:
        cursor = await db.execute(query, params)
        rows = await cursor.fetchall()
        return rows
```

#### Stream Queries

We've learned about `async for` and asynchronous generators in the previous step.
Here, we made use of async generators to implement stream queries.

There are broadly two categories of `async` queries we've used to fetch data in this service.
The first is a regular SELECT query that awaits the execution of the query then returns the resulting rows:

##### marketsvc/db_accessor.py

```py
async def execute_query(query, params):
    async with aiosqlite.connect(DB_PATH) as db:
        cursor = await db.execute(query, params)
        rows = await cursor.fetchall()
        return rows
```

This is suitable for a query like `get_total_cost_of_an_order()` since we only expect one row as a result, respresenting the total cost.

The second allows us to stream results one at a time from the database.
This is useful for requests where we anticipate a large number of items to be fetched from the database, we can make use of async generators to load just one object at a time in memory.

##### marketsvc/db_accessor.py

```py
async def stream_query(query, *params):
    async with aiosqlite.connect(DB_PATH) as db:
        async with db.execute(query, *params) as cursor:
            async for row in cursor:
                yield row
```

For example, in `/api/customers`, we can potentially have a huge number of customers.
So `get_customers()` calls `stream_query()` and returns its async generator that we can then consume from, in `server.py` to return a response.

In a real life example, this is helpful because you can then perform an aggregation on one object at a time without having to load all objects in memory at once to perform your aggregation.

#### Task Groups

You'll also notice, there's a new request `/api/orders_total` which calculates the total cost of a list of given order ids.
This request exercises `asyncio.TaskGroup()` which we've learned about in the previous step.
It kicks off a number of concurrent tasks `get_total_cost_of_an_order()` for each order.
When they are done, it returns a list of the results of all the tasks.

What happens if one of the queries raises an exception?

> _NOTE_
>
> While `/api/orders_total` can be implemented more efficiently using a single SQL query, we are using it as a way to allow you to exercise `TaskGroup()`.
> In a real life example, you will have other tasks to perform where you can truly benefit from concurrency.

### server.py

Finally, let's take a look at `server.py`.
You'll notice here that request handler functions with an async implementation now use the `async` keyword.

For requests that don't require large amounts of data to be loaded in memory, like `/api/order_total`, we simply `await` the async version of the implementation, before we return the response.

But what happens in the case of requests like `/api/customers` where we used an async generator?
As we saw in `db_accessor.py`, `get_customers()` is not a coroutine itself.
But it returns an async generator.

So in `server.py`, we can use list comprehension using the `async for` syntax to create our final list.

> ##### Test Your Understanding
>
> Take a look at the request handler for `/api/customers`, what is the memory footprint?
{: .block-tip }

Feel free to stop here and play around with the service:

```sh
./run.sh run
```

## Adding SQLAlchemy

We now want to replace the vanilla `aiosqlite` library with `SQLAlchemy` so we can get the same benefits we got with our sync service.

First, we'll add back `sqlalchemy` to our `requirements.txt`:

##### marketsvc/requirements.txt

```txt
aiosqlite
fastapi
uvicorn[standard]
sqlalchemy
greenlet
ruff
```

### The Async Engine

Next, we want to create an async SQLAlchemy engine.
Like we did in step 2, create a new file `marketsvc/db/base.py` with the following contents:

##### marketsvc/db/base.py

```py
from sqlalchemy.ext.asyncio import create_async_engine

engine = create_async_engine("sqlite+aiosqlite:///marketdb", echo=True)
```

Do these steps look familiar?
There are two key differences here:

1. The dialect is now `aiosqlite`, so we use `sqlite+aiosqlite`.
2. We now use `create_async_engine()` to create an [`AsyncEngine`](https://docs.sqlalchemy.org/en/20/orm/extensions/asyncio.html#sqlalchemy.ext.asyncio.AsyncEngine). The option `echo=True` logs the SQL output to stdout.

### Executing Queries

Next, we'll update the functions that execute our queries to use SQLAlchemy.

For `execute_query()`, update it to the following:

##### marketsvc/db_accessor.py

```py
from db.base import engine
from sqlalchemy import text

async def execute_query(query, params=None):
    async with engine.begin() as conn:
        return await conn.execute(text(query), params)
```

What does this mean?

[`AsyncEngine.begin()`](https://docs.sqlalchemy.org/en/20/orm/extensions/asyncio.html#sqlalchemy.ext.asyncio.AsyncEngine.begin) is a useful context manager which, on `__aenter__()`, delivers an `AsyncConnection` with an `AsyncTransaction` established.
On `__aexit__()`, it commits the transaction and closes the connection.
This means that we do not need to explicitly call `conn.commit()` or `conn.close()`.
All that's left to do then is to `await` the execution of the statement, and return the results in the desired format.

For `execute_insert_query()`, update it to the following:

##### marketsvc/db_accessor.py

```py
async def execute_insert_query(query, params=None):
    async with engine.begin() as conn:
        result = await conn.execute(text(query), params)
        await conn.commit()
        return result
```

> ##### Test Your Understanding
>
> Have you spotted a redundant line in `execute_insert_query()`?
{: .block-tip }

Finally, for `stream_query()`, we'll use [`AsyncConnection.stream()`](https://docs.sqlalchemy.org/en/20/orm/extensions/asyncio.html#sqlalchemy.ext.asyncio.AsyncConnection.stream) which allows us to load one `AsyncResult` at a time in memory, and asyncronoushly loop over the results:

##### marketsvc/db_accessor.py

```py
async def stream_query(query, params=None):
    async with engine.begin() as conn:
        result = await conn.stream(text(query), params)
        async for row in result:
            yield row
```

Now that we've updated the query execution functions, we just need to make a few updates to the way we fetch results for SQLAlchemy rows, as we'd done in `step-2-sqlalchemy`.

For example, `get_total_cost_of_an_order()` we fetch the result as follows:

##### marketsvc/db_accessor.py

```py
async def get_total_cost_of_an_order(order_id):
    result = ...
    return result.one().total
```

Alternatively, you can just checkout `step-6-asyncio-sqlalchemy-solved`.

Now, we are ready to run the service and play around with it.


> ##### Kudos!
>
> 🙌 You have now reached the `step-6-asyncio-sqlalchemy-solved` part of the tutorial. If not, checkout that branch and continue from there:
>```sh
>git checkout step-6-asyncio-sqlalchemy-solved
>```
{: .block-tip }

&nbsp;
