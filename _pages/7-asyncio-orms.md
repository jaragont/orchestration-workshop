---
title: SQLAlchemy ORMs with asyncio
author: Aya Elsayed, Rhythm Patel
category: SQLAlchemy Workshop
date: 2024-01-08
layout: post
---

Now that we've learned how to use SQLAlchemy with `asyncio`, it's time to bring back our ORMs.

To start, checkout the branch `step-7-asyncio-orms`:

```sh
git checkout step-7-asyncio-orms
```

This essentially builds on the work we've done in `step-3-orms` but uses the async engine we added in the previous step.

So in this step, we'll learn how to use the async version of [the 2.0 style queries](https://docs.sqlalchemy.org/en/14/glossary.html#term-2.0-style) we learned about in `step-3-orms`.

To do this, we'll use [`AsyncSession`](https://docs.sqlalchemy.org/en/14/orm/extensions/asyncio.html#sqlalchemy.ext.asyncio.AsyncSession).

So let's start in `db/base.py` add the following imports:

```py
from sqlalchemy.ext.asyncio import create_async_engine, AsyncAttrs
from sqlalchemy.ext.asyncio import async_sessionmaker
```

Next, replace the `engine` definition with:

```py
def async_session_maker():
    engine = create_async_engine(url_object, echo=True)
    return async_sessionmaker(engine, expire_on_commit=False)
```

This returns an `AsyncSession` object that we can use in `db_accessor.py`.

Finally, we need to update our `Base` class to use the `AsyncAttrs` mixin.
This mixin allows us to access attributes of any ORM class as an awaitable through the `.awaitable_attrs` attribute.
This is useful for attributes with lazy/deferred loading, as accessing the attribute directly will mean an implicit IO call to the database.
To prevent implicit IO calls with `asyncio`, we access lazily-loaded attributes as awaitables like so:

```py
await my_obj.awaitable_attrs.my_attr
```

Note that SQLAlchemy will throw an exception if an implicit IO call is made.

We elected to use `expire_on_commit=False` so we can access queried objects after a `session.commit()`

> ##### WARNING
> 
> The AsyncSession object is a mutable, stateful object which represents a single, stateful database transaction in progress.
> Using concurrent tasks with `asyncio`, with APIs such as `asyncio.gather()` for example, should use a separate AsyncSession per individual task.
> See [this link](https://docs.sqlalchemy.org/en/20/orm/extensions/asyncio.html#using-asyncsession-with-concurrent-tasks) for more info.
{: .block-warning }


## Updating the DB Queries

Now we are ready to update our queries.

Let's start with `get_customers()`.

First, we replace the `Session` with an `AsyncSession`:

```py
async def get_customers():
    async_session = async_session_maker()
    async with async_session() as session:
```

Using the session's context manager means we do not have to explicitly call session.close() when we are done.

Next, since we want to stream one customer object at a time rather query them all at once, we'll use the `stream_scalars()` API:

```py
        async for customer in await session.stream_scalars(stmt):
            yield customer.as_dict()
```

> ##### Test Your Understanding
>
> `customer.as_dict()` accesses `customer.address`. Why does SQLAlchemy not complain that we do not await the load of `customer.address`?
>
> _HINT_: what type of loading is used for `address` in the `Customer` ORM class?
{: .block-tip }


Next, let's update `get_total_cost_of_an_order()`.
As we don't need to stream a series of results here, all we need to do is just use the `async` version of the session and `await` the `execute` statement:

```py
async def get_total_cost_of_an_order(order_id):
    async_session = async_session_maker()
    async with async_session() as session:
        result = await session.execute(
            select(func.sum(Item.price * OrderItems.quantity).label("total_cost"))
            .join(Orders.order_items)
            .join(OrderItems.item)
            .where(Orders.id == order_id)
        )
        return result.scalar()
```

Finally, we'll walk you through updating `insert_order_items()`.
Again, we use the `async` version of the `session`.
What operations do you think we need to `await` here?

- `session.execute()`
- `session.flush()`
- `session.commit()`

Continue the rest of the queries now.

When you're done, your branch should match `step-7-asyncio-orms`

We are all done now!
