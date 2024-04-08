---
title: Optimisation Techniques
author: Aya Elsayed, Rhythm Patel
category: SQLAlchemy Workshop
date: 2024-01-05
layout: post
---

Let's talk about some techniques that will help to optimise your code when using ORMs with SQLAlchemy. You can checkout the `step-3-orms` branch to follow along this interactive section.

## Relationship Loading Techniques

SQLAlchemy provides extensive configuration over how related objects mapped using `relationship()` get loaded when querying. We can configure this by using [`relationship.lazy`](https://docs.sqlalchemy.org/en/20/orm/relationship_api.html#sqlalchemy.orm.relationship.params.lazy) parameter of `relationship()`.

There are 3 categories of relationship loading techniques: **lazy**, **eager**, and **no** loading.

### Lazy Loading

[Lazy loading](https://docs.sqlalchemy.org/en/20/orm/queryguide/relationships.html#lazy-loading) occurs when objects are returned from a query without loading the related objects initially. When the related object is first accessed, additional **SELECT** statements are emitted to access that particular related object. This is the default loading technique when using SQLAlchemy ORMs.

Thus in lazy loading, for a Parent and Child relationship, if we perform `select(Parent)`, only 1 **SELECT** statement will be emmitted and data will be fetched for attributes of `Parent`. If we then try to access the child, say `print(parent.child)`, another **SELECT** statement will be emmitted to fetch the attributes of `Child`.

Lazy loading is helpful when you know that you will not use all the related or child objects, or even if you will not require them instantly. It will also be beneficial on the "collection side" of a many-to-one relationship. Otherwise, if you use eager loading, SQLAlchemy will try to **JOIN** the tables to fetch all data at once, which can be expensive. We'll discuss eager loading in detail in a bit.

We've used eager loading for the `Customer` and `Address` relationship. In order to configure lazy loading, use `lazy="select"` in the `relationship()` construct. Let's convert it to lazy loading to see what effect it can have when fetching all customers.

##### db/customer.py

```py
class Customer(Base):
    ...

    address: Mapped["Address"] = relationship(
        back_populates="customer", lazy="select"
    )  # one to one
```

##### db/address.py

```py
class Address(Base):
    ...

    customer: Mapped["Customer"] = relationship(
        back_populates="address",
        lazy="select",
    )  # one to one
```

Let's add a `print` statement in `get_customers()` to access the related object, `Address`, for each of our customers within the session.

```py
def get_customers():
    with Session(engine) as session:
        stmt = select(Customer)
        result = session.execute(stmt)
        customers = result.scalars().all()
        for customer in customers:
            print(customer.address)

        return customers
```

##### SQLAlchemy logs and standard output

```sql
BEGIN (implicit)
SELECT customer.id, customer.name, customer.address_id FROM customer
[generated in 0.00044s] {}
SELECT address.id AS address_id, address.flat_number AS address_flat_number, address.post_code AS address_post_code FROM address WHERE address.id = %(pk_1)s
[generated in 0.00029s] {'pk_1': 1}
SELECT address.id AS address_id, address.flat_number AS address_flat_number, address.post_code AS address_post_code FROM address WHERE address.id = %(pk_1)s
[cached since 0.002041s ago] {'pk_1': 2}
SELECT address.id AS address_id, address.flat_number AS address_flat_number, address.post_code AS address_post_code FROM address WHERE address.id = %(pk_1)s
[cached since 0.003015s ago] {'pk_1': 3}
ROLLBACK
```

Here, the first **SELECT** statement only fetches the attributes linked to `Customer` i.e. `id`, `name`, and `address_id`. We then have 3 more **SELECT** statements to fetch address data for a particular `address_id` as seen with the **WHERE** clause and the bound parameter. This occurs for **each** customer.

> ##### What if?
>
> What happens if there are 100 customers? What about 1000? Or, what if we have 100,000 customers? Some real world services have millions of customers. How many **SELECT** statements would be emitted to fetch customer and address information?
{: .block-warning }

This is popular problem known as the **N+1 problem**. Initially, one **SELECT** statement is emitted to loads the result collection of parent objects. Then, for each parent object, an additional **SELECT** is emitted to load attributes of each related child object. Consequently, for N parent objects, we would have **N + 1 SELECT** statements.

> ##### Test Your Understanding
>
> For this scenario to fetch customer information, we require the `Address` information of all customers. Lazy loading is not a good strategy here. It becomes beneficial when we know we wouldn't be needing all the child objects. Is there a place in our service where lazy loading might be helpful?
{: .block-tip }


### Eager Loading

We can fix the N+1 problem by using **eager loading**. It refers when the related objects are loaded with the parent object up front. The child objects are automatically loaded along with its parent object.

The ORM accomplishes this by consolidating a **JOIN** to the **SELECT** statement to load the related objects at simultaneously, or by emitting additional **SELECT** statements to load the related child objects.

Eager loading is advantageous when we want to fetch information about all or numerous child objects along with its parent object. It's also a good practice to use this to reduce further queries and alleviate further load to the database.

As a result, eager loading is the right choice for `Customer` and `Address` relationship.

[Joined Eager Loading](https://docs.sqlalchemy.org/en/20/orm/queryguide/relationships.html#joined-eager-loading) is the most prominent type of eager loading. It applies a **JOIN** to the given **SELECT** statement to that related objects are loaded in the same result.

Let's update the `relationship()` in both `Customer` and `Address` to `lazy="joined"`, and run the request to get the customer information.

##### SQLAlchemy Logs

```sql
BEGIN (implicit)
SELECT customer.id, customer.name, customer.address_id, address_1.id AS id_1, address_1.flat_number, address_1.post_code 
FROM customer LEFT OUTER JOIN address AS address_1 ON address_1.id = customer.address_id
[generated in 0.00020s] {}
ROLLBACK
```

Here, you can see how we only emit 1 query to the database. The query has a **JOIN** to join the two tables and fetch all related information at once. We've also successfully avoided the N+1 problem.

### No Loading

No loading means to disable loading on a relationship. The child object is empty and is never loaded, or an error is raised with its accessed. No loading is used to prevent unwanted lazy loads.

> ##### Try It Yourself
>
> There are more [relationship loading techniques](https://docs.sqlalchemy.org/en/20/orm/queryguide/relationships.html), but the ones described above are the most popular. Try experimenting with different queries yourself and see the generated SQL statements in the logs.
{: .block-tip }


