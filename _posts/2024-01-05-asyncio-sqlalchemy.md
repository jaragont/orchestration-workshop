---
title: SQLAlchemy with asyncio
author: Aya Elsayed
category: SQLAlchemy Workshop
date: 2024-01-05
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

Let's read through a few of the requests.

### 1. /api/customers

