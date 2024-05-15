---
title: Getting Started
author: Aya Elsayed, Rhythm Patel
category: SQLAlchemy Workshop
date: 2024-01-02
layout: post
---

## Overview

Let's explore the files in `Marketplace`.
`Marketplace` is a `FastAPI` Python microservice that manages a `SQLite` database.
It retrieves information about customers, orders and can add new orders.

Our Python application directory is `marketsvc/`.
All the changes we'll be making in this tutorial will be in the `marketsvc/` folder.

### marketdb

Let's have a look at the [`SQLite`](https://www.sqlite.org/) database named `marketdb`.

Take a look at the SQL script `marketsvc/db/init_db.sql`.
It initialises our database with some tables and sample data.

For the purposes of this workshop, on service startup, the service executes this script to populate the database with sample data and reset it's state.
Take a look at it's contents to understand what tables to expect and the relationships between them.

`marketsvc/db/init_db.py` is a Python file that actually executes the aforementioned SQL script with SQLite.

### marketsvc

Our service consists of two layers: the API layer and the database interface layer.

#### The API layer

The API layer is implemented in `marketsvc/server.py`, which runs a `FastAPI` server that handles all our incoming HTTP requests.
For each API endpoint, it's request handler function uses the relevant functions from the database interface layer to prepare the appropriate response.

```py
app = FastAPI(debug=True)
```

With this line of code, we've initialized a FastAPI application.
Setting `debug=True` prints insightful logs to `stdout`.

```py
uvicorn.run("server:app", host="127.0.0.1", port=9090, reload=True)
```

Here, we are using `uvicorn` to customise the way our `FastAPI` app is run.
We've chosen to run our `app` on `127.0.0.1` (localhost) and listen to HTTP traffic on port `9090`.
We've also enabled the hot reload option which makes development easier.

#### The database interface layer

`marketsvc/db_accessor.py` is the accessor layer to the database, where we connect and retrieve data from the database.
You can see a number of functions that query or update the database with the relevant information.
Currently, they are using the `sqlite3` library and make raw SQL queries to SELECT/INSERT into the database.
We will be updating those queries over the course of this workshop.

Have a look at these functions to familiarise yourself with the queries they are performing.

## Installing Dependencies

With your virtual environment activated, let's install the required dependencies.
To do this, run:

```sh
python3.12 -m pip install -r requirements.txt
```

This will install the packages listed in `requirements.txt`.

You will need to repeat this step every time you add a new package to `requirements.txt`.

To verify that `IntelliSense` now works correctly, open up any `.py` file in your project.
`cmd`+click or `Ctrl`+click on any import in your file, you should be taken to the source file of the corresponding package.

## Running our services

Now that we know what to expect, let's run our service!

In the current terminal window, run:

```sh
python3.12 marketsvc/server.py
```

or you can use the convenience shell script `run.sh` to run the same command:

```sh
./run.sh run
```

You should see log a message like this once the server is running:

```sh
Uvicorn running on http://127.0.0.1:9090 (Press CTRL+C to quit)
Started reloader process [6523] using WatchFiles
Started server process [6531]
Waiting for application startup.
Application startup complete.
```

Our FastAPI app is now ready for requests.
Let's test it by running the following in a **new terminal window**:

```sh
curl -v http://localhost:9090/
```

You should see the text `Welcome to Marketplace!`.

You can also test out any of the other commands.
For example, we can retrieve a customer's orders by running:

```sh
curl http://localhost:9090/api/orders?cust_id=1
```

This should return a list of items in all of customer 1's orders.
Now that we understand the codebase, it's time to start with SQLAlchemy!

> ##### Tip
>
> Instead of writing out `curl` commands manually, we have provided a quick shell script for you `run.sh`.
> Take a look at its contents to see all the supported commands.
> For example, the curl command above can be run by running: `./run.sh custorders 1`
{: .block-tip }

&nbsp;
