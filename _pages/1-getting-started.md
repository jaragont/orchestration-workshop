---
title: Getting Started
author: Aya Elsayed, Rhythm Patel
category: SQLAlchemy Workshop
date: 2024-01-02
layout: post
---

## Overview

Let's explore the files in `Marketplace`.
Our Python application directory is `marketsvc/`.
All the changes we'll be making in this tutorial will be in the `marketsvc/` folder.

For this project, we'll be using a `SQLite` database and `Python` microservice with `FastAPI`.

### marketdb

Let's start by having have a look at the [`SQLite`](https://www.sqlite.org/) database named `marketdb`.

`marketsvc/db/init_db.sql` is a file that will initialise our database with some tables and sample data. 
It first drops all the tables to enforce a clean database state, creates all tables with the required columns and data types, and inserts some sample data in the tables. 
Feel free to have a look at it and understand the relationships between the tables.

`marketsvc/db/init_db.py` is a Python file that actually executes the aforementioned SQL script with SQLite.

### marketsvc

There are 2 main layers of our service: server and accessor.

#### server.py

`marketsvc/server.py` is the FastAPI server that handles all our incoming HTTP requests.
It routes a request with a particular endpoint to the designated function in db_accessor.

```py
app = FastAPI(debug=True)
```
With this line of code, we've initialized a FastAPI application.
Setting `debug=True` will give us some insights in the terminal window.

```py
uvicorn.run("server:app", host="127.0.0.1", port=9090, reload=True)
```
This allows our `app` to run on `127.0.0.1` (localhost) and the `9090` port.
We've also enabled the hot reload option which makes development easier.

#### db_accessor.py
 
`marketsvc/db_accessor.py` is the accessor layer to the database, where we connect and retrieve data from the database.
You can see a number of functions that query or update the database with the relevant information.

Have a look at these functions and the SQL queries in them.
The functions to actually execute these queries include a standard way of executing queries with SQLite.

## Installing Dependencies

Assuming you're in the virtual environment, let's install the required dependencies. To do this, run:

```sh
python3.12 -m pip install -r requirements.txt
```

This will install the packages listed in `requirements.txt`.

You will need to repeat this step every time you add a new package to `requirements.txt`.

Additionally, to verify that IntelliSense now works correctly, open up any `.py` file in your project.
`cmd`+click or `Ctrl`+click on any import in your file, you should be taken to the source file of the corresponding module.

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

You should see the heading `<h1>Welcome to Marketplace!</h1>`.

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
