---
title: Using ORMs
author: Aya Elsayed, Rhythm Patel
category: SQLAlchemy Workshop
date: 2024-01-04
layout: post
---

In the previous step, we've added SQLAlchemy to our codebase, but we still haven't fully leveraged its power. In this step, we'll create ORM classes to represent database tables.

## ORM

ORM stands for Object Relational Mapping, which lets us map the tables of a database to user-defined Python classes. The class attributes of the mapped class will be linked to columns of the table. 
This structure is called **Declarative Mapping**, which simultaneously defines a Python object model and metadata describing the actual tables that exist in a particular database.

Let's edit `db/base.py` to add a `Base` class that subclasses `DeclarativeBase`.

```py
from sqlalchemy.orm import DeclarativeBase

class Base(DeclarativeBase):
    pass
```

When we create new classes that inherit `Base`, they will each be established as **ORM mapped classes** at the creation time. Each mapped class will refer to a particular database table.

### Mapped Classes

The foundational unit of SQL when using the ORM is the Mapped Class. As we discovered, individual mapped classes are created by subclassing `Base`.

With this in mind, let's create an mapped class for `Customer` in `db/customer.py`.

```py
from db.address import Address
from db.base import Base
from sqlalchemy import ForeignKey
from sqlalchemy.orm import Mapped, mapped_column, relationship


class Customer(Base):
    __tablename__ = "customer"

    id: Mapped[int] = mapped_column(primary_key=True, autoincrement=True)
    name: Mapped[str]
    address_id: Mapped[int] = mapped_column(ForeignKey("address.id"))

    address: Mapped["Address"] = relationship(
        back_populates="customer", lazy="joined"
    )  # one to one
```

We indicate the name of the table by using the `__tablename__` class-level attribute.

We declare the columns by adduing them as attributes along with a special type called [`Mapped`](https://docs.sqlalchemy.org/en/20/orm/internals.html#sqlalchemy.orm.Mapped). The name of each attribute corresponds to the name of the column in the table. The datatype is inferred from the `Mapped` type, such as `int` becomes `INTEGER`, `str` becomes `VARCHAR`, etc. For example: `name: Mapped[str]` is the `name` column in the `customer` table of datatype `VARCHAR`.



> ##### TIP
>
> There's a useful SQLAlchemy extension [`Automap`](https://docs.sqlalchemy.org/en/20/orm/extensions/automap.html#) that automatically generates mapped classes for you.
> You can also use a mix of programmatic and auto-generated classes.
{: .block-tip }
