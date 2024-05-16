---
title: Adding SQLAlchemy
author: Aya Elsayed, Rhythm Patel
category: SQLAlchemy Workshop
date: 2024-01-03
layout: post
---

In the last step, we had seen that `db_accessor.py` used `sqlite3`, the SQLite database adapter, to interact with the database.
In this section, we will introduce SQLAlchemy, whose core will act as an abstraction layer to connect with the SQLite database.

Before we start making any code changes, we need to install `sqlalchemy` from [PyPI](https://pypi.org/).
In your `requirements.txt` file, add `sqlalchemy`:

##### requirements.txt

```txt
fastapi
uvicorn[standard]
sqlalchemy
ruff
```

Since we've added a new dependency, we need to install the new module using `pip`.
In a terminal window where your `venv` is active, run the following:

```sh
python3.12 -m pip install -r requirements.txt
```

Now, we're ready to start making code changes.

## Engine

Let's create `db/base.py` in the `marketsvc` folder to keep the database connection configuration in one place.

Here we create the [`Engine`](https://docs.sqlalchemy.org/en/20/core/engines.html) object, which is the entry point of any SQLAlchemy application.
The `Engine` serves as the main source of DBAPI connections to a given database.

##### marketsvc/db/base.py

```py
from sqlalchemy import create_engine

engine = create_engine("sqlite+pysqlite:///marketdb", echo=True)
```

We create the `Engine` with the `create_engine()` method. Here, we have passed in a string to `create_engine()`, which includes all the necessary information required to connect to a database:

1. Type of database, in our case, `sqlite`.
    This instructs SQLAlchemy to use the SQLite [**dialect**](https://docs.sqlalchemy.org/en/20/glossary.html#term-dialect), which is a system used to communicate with various kinds of databases and their respective drivers.
2. DBAPI, in our case, `pysqlite`.
    [**DBAPI**](https://docs.sqlalchemy.org/en/20/glossary.html#term-DBAPI) (Python Database API Specification) is a driver that SQLAlchemy uses to connect to a database.
    It's a "low level" API that lets Python talk to the database.
3. The name of the database, in our case, `marketdb`

We've also enabled the `echo` flag, which will log the generated SQL queries by SQLAlchemy.
We'll display these logs to explain what happens behind the scenes.

> **Lazy Connecting**
>
> When `create_engine()` first returns an `Engine` object, it will not reach out to the database yet.
> It will wait for a task to be performed against the database, such as a SELECT query, and then connect to the database, following the [lazy initialization](https://en.wikipedia.org/wiki/Lazy_initialization) design pattern.

## Connection

Once our `Engine` object is ready, it will be used to connect to the database by providing a unit of connectivity called the [`Connection`](https://docs.sqlalchemy.org/en/20/core/connections.html).

`Connection` is a proxy object for an actual DBAPI connection, and provides services to execute SQL statements and transaction control, and this is how we'll interact with the database.
The `Connection` is acquired by `Engine.connect()`.

We don't want to keep a `Connection` running indefinitely, and thus, the recommended way to use `Connection` is with context managers, which will frame the operations inside into a transaction, and will invoke `Connection.close()` at the end of the block.

## Executing Queries

Now that we understand how to connect to the database, let's look at how to execute queries with SQLAlchemy.
For now, we'll be using raw SQL.
Let's update `execute_query()` in `db_accessor.py` to use the `Connection` context manager:

##### marketsvc/db_accessor.py

```py
from db.base import engine
from sqlalchemy import text

def execute_query(query, params=None):
    with engine.connect() as conn:
        return conn.execute(text(query), params)
```

The function `get_customers()`, for example, in `db_accessor.py` calls `execute_query()` as follows:

```py
def get_customers():
    rows = execute_query("SELECT * FROM customer")
    return rows
```

The statement is executed with the [`Connection.execute()` function](https://docs.sqlalchemy.org/en/20/core/connections.html#sqlalchemy.engine.Connection.execute), which returns a [`Result`](https://docs.sqlalchemy.org/en/20/core/connections.html#sqlalchemy.engine.Result); an iterable object of resulting [`Row`](https://docs.sqlalchemy.org/en/20/core/connections.html#sqlalchemy.engine.Row)s.

In order to format the response correctly, let's edit the `customers()` function in `server.py` to get the `Row` objects as dictionaries.

##### marketsvc/server.py

```py
@app.get("/api/customers")
def customers():
    customers = get_customers()
    return (customer._asdict() for customer in customers)
```

At this point, the API to fetch customers is updated to use SQLAlchemy.
Run the service:

```sh
./run.sh run
```

Now, in another shell, hit the `/api/customers` endpoint using `curl`. or use our convenience shell script:

```sh
./run.sh customers
```

##### SQLAlchemy logs

```sql
BEGIN (implicit)
SELECT * FROM customer
[generated in 0.00021s] ()
ROLLBACK
```

As you can see from the logs, a **ROLLBACK** was emitted at the end.
This marked the of the transaction.
An automatic rollback occurs when a connection is closed after use, to ensure that the connection is 'clean' for its next use.

> ##### Test Your Understanding
>
> Now that you understand how SELECT queries work in SQLAlchemy, are there any other `@app.get` endpoints in our service that require updates to their `server.py` or `db_accessor.py` handlers?
> If you are stuck, take a look at the branch `step-2-sqlalchemy` for a hint!
>
{: .block-tip }

## Parameter Binding

We may want to select specific rows, or insert some data to the table.
The `Connection.execute()` function can accept parameters called [**bound parameters**](https://docs.sqlalchemy.org/en/20/glossary.html#term-bound-parameters).
We indicate the presense of parameters in the `text()` construct by using colons, such as `:customer_id`.
We can then send the actual value of these parameters as a dictionary in the second argument of `Connection.execute()`; for example:

```py
conn.execute(
    text("SELECT * FROM customer WHERE id=:customer_id"),
    {"customer_id": 1},
)
```

> ##### WARNING
>
> Never put variables directly in the query string.
> Doing so leaves your code vulnerable to SQL injection attacks.
> **Always** use parameter binding.
> Using parameters allows the dialect and DBAPI to correctly handle the input, and enables the driver to have the best performance.
{: .block-danger }

If we want to send multiple sets of parameters, such as insert multiple records in the table, we can pass a **list of dictionaries** to `Connection.execute()` and send multiple parameter sets. The SQL statement will be executed once for each parameter set.

## Committing Data

In this section, we'll learn about two commit styles.

### Commit As You Go

You might have noticed the absense of the **COMMIT** statement from the SQLAlchemy logs.
If we want to commit some data, we need to explicitly call [`Connection.commit()`](https://docs.sqlalchemy.org/en/20/core/connections.html#sqlalchemy.engine.Connection.commit) inside the block.

Let's update `execute_insert_query()` to invoke `Connection.commit()`.

##### marketsvc/db_accessor.py

```py
def execute_insert_query(query, params):
    with engine.connect() as conn:
        cursor = conn.execute(text(query), params)
        result = cursor.fetchone()
        conn.commit()
        return result
```

Here, we've used the resulting `Cursor` object from the `execute` statement to fetch a single result, as we expect our statement to result in exactly one result.
This result is in the form of a Python `namedtuple`.

Therefore, we can update `add_new_order_for_customer()` to fetch the `id` of the new order by using `.id`:

```py
def add_new_order_for_customer(customer_id, items):
    try:
        new_order_id = execute_insert_query(
            """
            INSERT INTO orders
                (customer_id, order_time)
            VALUES
                (:customer_id, Date('now'))
            RETURNING id
            """,
            {"customer_id": customer_id},
        ).id
        
        ...
```

If you were to run `./run.sh neworder` to exercise `add_new_order_for_customer()` at this point, where you insert a row in the `orders` table, the logs generated by SQLAlchemy would look like as follows:

##### SQLAlchemy logs:
```sql
BEGIN (implicit)
INSERT INTO orders (customer_id, order_time) VALUES (?, Date('now')) RETURNING id
[generated in 0.00041s] (1,)
COMMIT
```

As you can see, `conn.commit()` committed the transaction, and the statement **COMMIT** is logged, as compared to **ROLLBACK** in the previous example.
We can then call `conn.commit()` for committing additional statements. This style is called **commit as you go**.

However, if an exception occurs duing the transaction, the changes will be rolled back and a **ROLLBACK** will be displayed instead.

> ##### Test Your Understanding
>
> You can see that `add_new_order_for_customer()` has two INSERT statements.
> The first adds a new order to the `orders` table, and the second adds the items of the new order into the `order_items` table, and associates them with the `new_order_id` we just created.
> You may have noticed that you only see the SQL logs for the first INSERT statement but not the second one.
> Why is that?
>
{: .block-tip }

Next, let's update `execute_insert_queries()` to use SQLAlchemy:

##### SQLAlchemy logs:
```py
def execute_insert_queries(query, params_tuple):
    with engine.connect() as conn:
        conn.execute(text(query), params_tuple)
        conn.commit()
```

Now if you exercise `add_new_order_for_customer()` again, you should see the logs for both INSERT statements.

Note here that the difference in the implementation between `execute_insert_query()` and `execute_insert_queries()` is that in the former, we fetch results from the cursor.

Because we've used the syntax `RETURNING id` in the first INSERT statement, `SQLAlchemy` expects the results to be fetched before the `connection.commit()` is called.
If you were to skip the call to `result = cursor.fetchone()` when you execute the first INSERT statement, SQLAlchemy would throw an exception:

```txt
sqlalchemy.exc.OperationalError: (sqlite3.OperationalError) cannot commit transaction - SQL statements in progress
```

### Begin Once

Another way to commit data is to use the context manager `Engine.begin()` instead of `Engine.connect()`.
It will declare the whole block to be one transcation block, and will enclose everything inside the transaction with one **COMMIT** at the end.
This method is called **begin once**.

```py
with engine.begin() as conn:
    result = conn.execute(text("INSERT INTO orders..."), params)
```

##### SQLAlchemy logs

```sql
BEGIN (implicit)
INSERT INTO orders ...
[generated in 0.00041s] (...)
COMMIT
```

Notice there is a **COMMIT** without explicitly writing `conn.commit()`.

&nbsp;

Now that we've added SQLAlchemy, let's eliminate raw SQL text and introduce ORMs!

> ##### Kudos!
>
> 🙌 You have now reached the `step-2-sqlalchemy` part of the tutorial. If not, checkout that branch and continue from there:
>```sh
>git checkout step-2-sqlalchemy
>```
{: .block-tip }

&nbsp;
