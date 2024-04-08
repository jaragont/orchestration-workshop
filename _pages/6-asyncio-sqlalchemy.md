---
title: SQLAlchemy with asyncio
author: Aya Elsayed, Rhythm Patel
category: SQLAlchemy Workshop
date: 2024-01-07
layout: post
---

In the last step, we learned about `asyncio`.
Now let's see how we can use SQLAlchemy with an async service.
To start, checkout the branch `step-6-asyncio-asyncpg`:

```sh
git checkout step-6-asyncio-asyncpg
```

This is an async version of the service we say in `step-1-psychopg2`.
Except it uses the the [`async` flavour of `Flask`](https://flask.palletsprojects.com/en/3.0.x/async-await/) and the [postgres async package `asyncpg`](https://magicstack.github.io/asyncpg/current/).

Since we've installed new packages, let's add them to our `venv` so we can get intellisense for these packages.

```sh
python -m pip install -r marketsvc/requirements.txt
```

Now, let's read through a few key files.

### server.py

You'll notice here a number of requests that have an async implementation use the keyword `async`.

For requests where we anticipate a potentially large number of items to be returned, an async generator was used in the implementation.
For example, in `/api/customers`, `get_customers()` returns an async generator that we then consume to return a response.
In a real life example, you may want to opt for a multi-part response to your request so that you don't have to load all the contents in memory at once.

For requests that don't require large amounts of data to be loaded in memory, like `/api/order_total`, we simply `await` the async version of the implementation, before we return the response.

You'll also notice, there's a new request `/api/orders_total` which calculates the total cost of a list of given order ids.
This request exercises `asyncio.TaskGroup()` which we've learned about in the previous step.
It kicks off a number of concurrent tasks `get_total_cost_of_an_order()` for each order.
When they are done, it returns a list of the results of all the tasks.

### db_accessor.py

Let's start by looking at `get_customers()`.
As we saw in `server.py`, we expect an async generator here, so how do we achieve that?

Look at the implementation of `stream_query()`.
You'll notice that we've used `asyncpg`'s [`conn.cursor()`](https://magicstack.github.io/asyncpg/current/api/index.html#asyncpg.connection.Connection.cursor) here, which is compatible with the `async for` syntax.
We then yield one dictionary row at a time, which is what we end up sending as a json response.

> ##### Test Your Understanding
>
> Why is `get_customers()` a regular method not an `async` one?
{: .block-tip }

In contrast, let's look at `get_total_cost_of_an_order()`.
Since we just expect a single number here as a response, we await `execute_query()`, which is implemented using `asyncpg`'s [`conn.fetch()`](https://magicstack.github.io/asyncpg/current/api/index.html#asyncpg.connection.Connection.fetch) interface.
We simply await the query execution and return the sum.
While this on it's own is not a very useful async request, it becomes useful when you want to perform multiple concurrent actions, like in `/api/orders_total`.

Feel free to stop here and run the service:

```sh
docker compose build
docker compose up
```

## Adding SQLAlchemy

We now want to replace the vanilla `asyncpg` library with `SQLAlchemy` so we can get the same benefits we got with our sync service.

First, we'll add back `sqlalchemy` to our requirements.txt:

```txt
flask[async]
asyncpg
sqlalchemy
```

Next, we want to create an async SQLAlchemy engine.
Like we did in step 3, create a new file `marketsvc/db/base.py` with the following contents:

```py
import os
from sqlalchemy import create_engine, URL
from sqlalchemy.ext.asyncio import create_async_engine

DB_USER = os.environ.get("POSTGRES_USER")
DB_PASSWORD = os.environ.get("POSTGRES_PASSWORD")
DB_PORT = os.environ.get("POSTGRES_PORT")
DB_NAME = os.environ.get("POSTGRES_DB")
DB_HOST = "marketdb"

url_object = URL.create(
    "postgresql+asyncpg",
    username=DB_USER,
    password=DB_PASSWORD,
    host=DB_HOST,
    database=DB_NAME,
    port=DB_PORT,
)


def create_engine():
    return create_async_engine(url_object, echo=True)
```

Do these steps look familiar?
There are two key differences here:

1. the dialect of postgres in the `URL` object is now `asyncpg`, so we use `"postgresql+asyncpg"`.
2. we now use `create_async_engine()` to create an [`AsyncEngine`](https://docs.sqlalchemy.org/en/20/orm/extensions/asyncio.html#sqlalchemy.ext.asyncio.AsyncEngine).
    `echo=True` enables log output.

Next, we'll update the functions that execute our queries to use SQLAlchemy.

For `execute_query()`, update it to the following:

```py
async def execute_query(query, params=None, insert=False):
    async with create_engine().begin() as conn:
        result = await conn.execute(text(query), params)
        return [row._asdict() for row in result]
```

What does this mean?

[`AsyncEngine.begin()`](https://docs.sqlalchemy.org/en/20/orm/extensions/asyncio.html#sqlalchemy.ext.asyncio.AsyncEngine.begin) is a useful context manager which, on `__enter__()`, delivers an `AsyncConnection` with an `AsyncTransaction` established.
This means that we need not explicitly call `conn.commit()` or `conn.close()` as the context manager takes care of that for us.
All that's left to do then is to await the execution of the statement, and return the results in the desired format.

For `execute_insert_query()`, update it to the following:

```py
async def execute_insert_query(query, params):
    async with create_engine().begin() as conn:
        await conn.execute(text(query), params)
```

Next, for the insert case, we now understand why this suffices.

Finally, for `stream_query()`, we'll use [`AsyncConnection.stream()`](https://docs.sqlalchemy.org/en/20/orm/extensions/asyncio.html#sqlalchemy.ext.asyncio.AsyncConnection.stream) which allows us to asyncronoushly loop over the results:

```py
async def stream_query(query, params=None, insert=False):
    async with create_engine().begin() as conn:
        result = await conn.stream(text(query), params)
        async for row in result:
            yield row._asdict()
```

Now that we've updated the query execution functions, we just need to update the SQL queries to use the syntax expected by SQLAlchemy.
As we've done this already in step 3, go ahead and copy the correct SQL queries from your `sync` branch into this file.

When you're done, your branch should match the results in `step-6-asyncio-sqlalchemy`.

&nbsp;
