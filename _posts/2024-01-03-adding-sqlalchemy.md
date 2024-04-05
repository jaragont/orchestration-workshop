---
title: Adding SQLAlchemy
author: Aya Elsayed
category: SQLAlchemy Workshop
date: 2024-01-03
layout: post
---

In the last step, we had seen that `db_accessor.py` used `psycopg2`, the PostgreSQL database adapter, to interact with the database.
In this section, we will introduce SQLAlchemy, whose core will act as an abstraction layer to connect with the PostgresSQL database.

Let's have a look at `db/base.py`.

Here we create the `Engine` object, which is the entry point of any SQLAlchemy application.  The `Engine` serves as the main source of DBAPI connections to a given database. We can create the `Engine` with the `create_engine()` method. Here, we have passed in a `URL` object to `create_engine()`, which includes all the necessary information required to connect to a database:
- Type of database, represented by `postgresql`. 
    - This instructs SQLAlchemy to use the PostgreSQL **dialect**, which is a system used to communicate with various kinds of databases and their respective drivers.
- DBAPI, represented by `psycopg2`.
    - **DBAPI** (Python Database API Specification) is a driver that SQLAlchemy uses to connect to a database. It's a "low level" API that lets Python talk to the database.
- Database connection configuration like host, port, database, username, and password.

We've also enabled the `echo` flag, which will log the generated SQL queries by SQLAlchemy.

> **Lazy Initialization**
> 
> When `create_engine()` first returns an `Engine` object, will not reach out to the database yet. It will connect the database the first time any task would be performed against the DB, such as a SELECT query. 
