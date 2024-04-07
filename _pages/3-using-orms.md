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

With this in mind, let's create a mapped class for `Customer` in `db/customer.py`.

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

    def __repr__(self) -> str:
        return f"Item(id={self.id!r}, name={self.name!r}, address_id={self.address_id!r}, address={self.address})"

    def as_dict(self):
        return {
            "name": self.name,
            "address": {
                "flat_number": self.address.flat_number,
                "post_code": self.address.post_code,
            },
        }
```

We indicate the name of the table by using the [`__tablename__`](https://docs.sqlalchemy.org/en/20/orm/mapping_api.html#sqlalchemy.orm.DeclarativeBase.__tablename__) class-level attribute.

#### Mapping Columns

We declare the columns by adding them as attributes along with a special type called [`Mapped`](https://docs.sqlalchemy.org/en/20/orm/internals.html#sqlalchemy.orm.Mapped), which is driven from [PEP 484](https://peps.python.org/pep-0484/). The name of each attribute corresponds to the name of the column in the table. The datatype is inferred from the `Mapped` type, such as `int` becomes `INTEGER`, `str` becomes `VARCHAR`, etc. 

For example: `name: Mapped[str]` is the `name` column in the `customer` table of datatype `VARCHAR`. Consequently, we can indicate nullable contraints in the column by `Mapped[str | None]`.

To include additional specification for our attributes, we can use the [`mapped_column()`](https://docs.sqlalchemy.org/en/20/orm/mapping_api.html#sqlalchemy.orm.mapped_column) construct, which will generate `Column` objects. We can `mapped_column()` without annotations too. For example, `customer_name = mapped_column("name", String, nullable=False)` is also valid. Notice how it's possible for the class attribute to have a different name to the column. The datatype of the column is inferred from the [String](https://docs.sqlalchemy.org/en/20/core/type_basics.html#sqlalchemy.types.String) SQLAlchemy type object.


We also indicate the presense of the primary key with enabling the `primary_key` boolean parameter. If there is a composite key constraint, we can represent that by enabling the `primary_key` boolean flag for all the attributes that are a part of the composite key.

In our code, we've set the `autoincrement` parameter to `True`. This is because our IDs are integers starting from 1, and incremented automatically, i.e., we do not need to assign an integer to `id` while inserting rows.

Similarly, we can depict foreign key constraint by using the [`ForeignKey`](https://docs.sqlalchemy.org/en/20/core/constraints.html#sqlalchemy.schema.ForeignKey) object in `mapped_class()`. The syntax is `<table_name>.<column_name>`. Thus, the `address_id` attribute in our `Customer` class is the foreign key to the `id` column in `address` table.

The classes include a default `__init__()` method that accepts all attribute names as optional keyword arguments. This will allow us to instantiate objects, which will help us to insert records in the table. We'll talk more about this later.

#### Relationships

In addition to he column-based attributes, we have a [`relationship()`](https://docs.sqlalchemy.org/en/20/orm/relationship_api.html#sqlalchemy.orm.relationship) construct that represents a linkage between two mapped classes, and by extension, relationship between two tables.

In our case, the `relationship()` construct, along with the `Mapped["Address"]` construct, will be used to inspect the table relationships. As a result, `Customer.address` links `Customer` to `Address`. We'll talk about its parameters in a bit. The relationship linkage between the two tables is inferred with the foreign key constraint we had defined previously.


Finally, we have a `__repr__()` method which is not necessary, but helps us in debugging. We also have a `as_dict()` method which converts the ORM class into a dictionary with a meaningful structure that will be used to return responses for our service.





> ##### TIP
>
> There's a useful SQLAlchemy extension [`Automap`](https://docs.sqlalchemy.org/en/20/orm/extensions/automap.html#) that automatically generates mapped classes for you.
> You can also use a mix of programmatic and auto-generated classes.
{: .block-tip }
