---
title: Getting Started
author: Aya Elsayed
category: SQLAlchemy Workshop
date: 2024-01-02
layout: post
---

Let's explore the files in our `MarketPlace`.

For this project, we'll be using `Docker` to run our `postgres` database and Python microservice.
So let's start at the [`compose.yaml`](https://docs.docker.com/compose/compose-application-model/#:~:text=The%20Compose%20file,prefers%20the%20canonical%20compose.yaml%20.) file.
This is the file that tells `Docker` which containers are part of our setup and how to configure them.

As you'd expect, there are two `docker` services:

### marketdb

`marketdb` is the container that runs our `postgres` db.

To use a `postgres` db, we use the [postgres docker image](https://hub.docker.com/_/postgres) which runs a `postgres` instance.

```yaml
image: postgres
env_file: .env.yaml
```

We set the relevant environment variables so we can interact with the database in the file `.env.yaml`.
Take a moment to look at its contents.

Next, we expose port `5432` which we've used as our `POSTGRES_PORT` to interact with the database.


```yaml
    ports:
    - 5432:5432
```

We mount a single file `marketdb/init_db.sql` at the `/docker-entrypoint-initdb.d/` path of our `postgres` container.
This is a file that will initialise our database with some tables and sample data on container start-up.
Take a look at the contents of `marketdb/init_db.sql` to familiarise yourself with the database structure.

```yaml
    volumes:
      - ./marketdb/init_db.sql:/docker-entrypoint-initdb.d/init_db.sql
```

Finally, we define a `healthcheck` to let other `docker` containers understand when the `marketdb` container is healthy and ready to be used.

### marketsvc

`marketsvc` is the container that runs our Python service.

In `marketsvc/Dockerfile`, you can see that it's simply a `Python3.12` image that installs the pip requirements listed in `requirements.txt`.

```Dockerfile
FROM python:3.12

WORKDIR /code
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
```

To start with, we only have two dependencies for our service, as it's a barebones `flask` service that uses `psycopg2` to query the database.
You can see how it's used in code at `db_accessor.py`.
You can see a number of functions that query or update the database with the relevant information.

Next, let's take a look at `server.py`, as you can see, it is a simple `flask` app that handles various requests it receives.
Finally, we can communicate with our flask app over port `9090` as you can see in its `compose.yaml` configuration:

```yaml
    ports:
    - 9090:9090
```

## Running our services

Now that we know what to expect, let's run our containers and give them a quick test.
First, let's build and run our docker containers.
From the repo root, run the following:

```sh
docker compose build
```

This tells `docker` to build the containers defined in the `compose.yaml` file.
When it's done building, run the following:

```sh
docker compose up
```

This brings up our two containers.
In a moment, you should be able to see that both containers have come up.
You can see that through the logs printed to stdout.

> _HINT:_ you can also use Codespaces' `Docker` extension to see which containers are running.

`marketsvc` should print something like this once it's ready:

```sh
marketsvc-1  |  * Running on http://127.0.0.1:9090
```

Our flask app is now ready for requests.
Let's test it by running the following:

```sh
curl -v http://localhost:9090/
```

You should see the heading `<h1>Welcome to MarketPlace!</h1>`.

You can also test out any of the other commands.
For example, we can retrieve a customer's orders by running:

```sh
curl http://localhost:9090/api/orders?cust_id=1
```

This should return a list of items in all of customer 1's previous orders.
Now that we understand the codebase, it's time to start with SQLAlchemy!
