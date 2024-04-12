---
title: SQLAlchemy ORMs with asyncio
author: Aya Elsayed, Rhythm Patel
category: SQLAlchemy Workshop
date: 2024-01-08
layout: post
---

Now that we've learned how to use SQLAlchemy with `asyncio`, it's time to bring back our ORMs.
We'll also look at how to use the async version of the SQL Expression language we learned about in `step-3-orms`.

To start, checkout the branch `step-7-asyncio-orms-base`:

```sh
git checkout step-7-asyncio-orms-base
```

This essentially builds on the work we've done in `step-3-orms` but uses the async engine we added in the previous step.

To start, we'll need to use an [`AsyncSession`](https://docs.sqlalchemy.org/en/14/orm/extensions/asyncio.html#sqlalchemy.ext.asyncio.AsyncSession).
Since we [need a dedicated session per connection](https://docs.sqlalchemy.org/en/20/orm/extensions/asyncio.html#using-asyncsession-with-concurrent-tasks), we'll use a factory to create our `AsyncSession`s.

To do this, we'll start in `db/base.py` add the following imports:

##### marketsvc/db/base.py

```py
from sqlalchemy.ext.asyncio import (
    AsyncAttrs,
    async_sessionmaker,
    create_async_engine,
)
```

Next, we create an `async_sessionmaker` to create our `AsyncSession` instances:

```py
engine = create_async_engine(url_object, poolclass=NullPool, echo=True)
async_session_maker = async_sessionmaker(engine, expire_on_commit=False)
```

This returns an `AsyncSession` object that we can use in `db_accessor.py`.

Finally, we need to update our `Base` class to use the `AsyncAttrs` mixin:

##### marketsvc/db/base.py

```py
class Base(AsyncAttrs, DeclarativeBase):
    pass
```

This mixin allows us to access attributes of any ORM class as an awaitable through the `.awaitable_attrs` attribute.
This is useful for attributes with lazy/deferred loading, as accessing the attribute directly will mean an implicit IO call to the database.
To prevent implicit IO calls with `asyncio`, we access lazily-loaded attributes as awaitables like so:

```py
await my_obj.awaitable_attrs.my_attr
```

Note that SQLAlchemy will throw an exception if an implicit IO call is in `async` mode.

For example, if `Customer` had a lazily-loaded relationship to `Address`.
To access `address`, I need to `await` another SELECT statement to be executed.

```py
address = await customer.awaitable_attrs.address
```

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

##### marketsvc/db_accessor.py

```py
async def get_customers():
    async_session = async_session_maker()
    async with async_session() as session:
```

Using the session's context manager means we do not have to explicitly call `session.close()` when we are done.

Next, since we want to stream one customer object at a time rather query them all at once, we'll use the `stream_scalars()` API:

##### marketsvc/db_accessor.py

```py
    async with async_session_maker() as session:
        stmt = select(Customer)
        async for customer in await session.stream_scalars(stmt):
            yield customer
```

> ##### Test Your Understanding
>
> In `server.py`, we call `customer.as_dict()` which accesses `customer.address`. Why does SQLAlchemy not complain that we do not await the load of `customer.address`?
>
> _HINT_: what type of loading is used for `address` in the `Customer` ORM class?
{: .block-tip }

Next, let's update `get_total_cost_of_an_order()`.
As we don't need to stream a series of results here, all we need to do is just use the `async` version of the session and `await` the `execute` statement:

##### marketsvc/db_accessor.py

```py
async def get_total_cost_of_an_order(order_id):
    async_session = async_session_maker()
    async with async_session() as session:
        result = await session.execute(
            select(func.sum(OrderItems.item_total_expression)
            .join(Orders.order_items)
            .join(OrderItems.item)
            .where(Orders.id == order_id)
        )
        return result.scalar()
```

Finally, for `insert_order_items()`, we use the `async` version of the `session`.
What operations do you think we need to `await` here?

- `session.execute()`
- `session.add()`
- `session.commit()`

##### marketsvc/db_accessor.py

```py
async def add_new_order_for_customer(customer_id, items):
    try:
        async with async_session_maker() as session:
            result = await session.execute(
                select(Customer).where(customer_id == customer_id)
            )
            customer = result.scalar()

            new_order = Orders(
                customer_id=customer_id,
                order_time=datetime.now(),
                customer=customer,
            )

            new_order.order_items = [
                OrderItems(
                    item_id=item["id"],
                    quantity=item["quantity"],
                )
                for item in items
            ]

            session.add(new_order)
            await session.commit()
        return True

    except Exception:
        logging.exception("Failed to add new order")
        return False
```

You may continue updating the rest of the queries to use the async version of the SQL expression language.

> ##### Kudos!
>
> 🙌 You are all done! You've reached the final step of the tutorial.
> Your code should now match `step-7-asyncio-orms-solved`
>```sh
>git checkout step-7-asyncio-orms-solved
>```
{: .block-tip }

&nbsp;
