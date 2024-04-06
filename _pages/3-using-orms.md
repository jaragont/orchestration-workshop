---
title: Using ORMs
author: Aya Elsayed, Rhythm Patel
category: SQLAlchemy Workshop
date: 2024-01-04
layout: post
---

In the last step, we've seen that `db_accessor.py` uses raw SQL queries to update the DB.
In this section, we will replace these raw queries with SQLAlchemy.

> ##### TIP
>
> There's a useful SQLAlchemy extension [`Automap`](https://docs.sqlalchemy.org/en/20/orm/extensions/automap.html#) that automatically generates mapped classes for you.
> You can also use a mix of programmatic and auto-generated classes.
{: .block-tip }
