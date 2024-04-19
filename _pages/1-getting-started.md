---
title: Getting Started
author: Aya Elsayed, Rhythm Patel
category: SQLAlchemy Workshop
date: 2024-01-02
layout: post
---

Let's explore the files in `Marketplace`.
Our Python application directory is `marketsvc/`.
All the changes we'll be making in this tutorial will be in the `marketsvc/` folder.

For this project, we'll be using `Docker` to run our `postgres` database and `Python` microservice.
So let's start at the [`compose.yaml`](https://docs.docker.com/compose/compose-application-model/#:~:text=The%20Compose%20file,prefers%20the%20canonical%20compose.yaml%20.) file.
This is the file that tells `docker` which containers are part of our setup and how to configure them.

As you'd expect, there are two `docker` services: `marketdb` and `marketsvc`.

### marketdb

`marketdb` is the container that runs our `postgres` db.

To use a `postgres` db, we use the [Postgres Docker image](https://hub.docker.com/_/postgres) which runs a `postgres` instance.

```yaml
image: postgres
env_file: .env.yaml
```

We set the relevant environment variables so we can interact with the database in the file `.env.yaml`.
Take a moment to look at its contents.

Next, since we've used port `5432` as our `POSTGRES_PORT` (see .env.yaml), we also need to tell `docker` to expose this port so that the database can receive traffic from outside the container through this port:

```yaml
    ports:
    - 5432:5432
```

Then, we mount a single file `marketdb/init_db.sql` at the `/docker-entrypoint-initdb.d/` path of our `postgres` container.
This is a file that will initialise our database with some tables and sample data on container start-up.
Take a look at the contents of `marketdb/init_db.sql` to familiarise yourself with the database structure.

```yaml
    volumes:
      - ./marketdb/init_db.sql:/docker-entrypoint-initdb.d/init_db.sql
```

Finally, we define a `healthcheck` to let other `docker` containers understand when the `marketdb` container is healthy and ready to be used.
We use the utility `pg_isready` to ping the database at a repeated interval until it reports that it's ready.

### marketsvc

`marketsvc` is the container that runs our `Python` service.

In `marketsvc/Dockerfile`, you can see that it's simply a `Python3.12` image that installs the pip requirements listed in `requirements.txt`.

```Dockerfile
FROM python:3.12

WORKDIR /code
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
```

To start with, we only have two dependencies for our service, as it's a barebones `flask` service that uses `psycopg2` to query the database.
You can see how `psycopg2` is used in `marketsvc/db_accessor.py`.
You can see a number of functions that query or update the database with the relevant information.

For `flask`, take a look at `server.py`, as you can see, it is a simple `flask` app that handles various requests it receives.
Finally, we can communicate with our flask app over port `9090` as you can see in its `compose.yaml` configuration:

```yaml
    ports:
    - 9090:9090
```

Finally, we tell `docker` what order the services need to be started in as `marketdb` needs to be ready first before we start `marketsvc`:

```yaml
    depends_on:
      marketdb:
        condition: service_healthy
```

## Installing Dependencies for Development

In the previous section, we learned that the `docker` container `marketsvc` will install the required dependencies in the `marketsvc` container to be able to run the service.

But to make our development experience better, it's useful to also install these dependencies in the `venv` of our `Codespaces` container.
In the previous step, we've created and activated a `venv`.
Now that we're familiar with the dependencies we will be using, let's install them in our `venv`:

```sh
python -m pip install -r marketsvc/requirements.txt
```

You will need to repeat this step every time you add a new package to `marketsvc/requirements.txt` to have `Codespaces` recognise it.

To verify that IntelliSense now works correctly, open up any `.py` file in your project.
`cmd`+click or `Ctrl`+click on any import in your file, you should be taken to the source file of the corresponding module.

## Running our services

Now that we know what to expect, let's run our containers and give them a quick test.
First, let's build and run our `docker` containers.
From the repo root, run the following:

```sh
docker compose build
```

This tells `docker` to build the containers defined in the `compose.yaml` file.
When it's done building, run the following:

```sh
docker compose run -p 9090:9090 marketsvc
```

or you can use the convenience shell script `run.sh` to run the same command:

```sh
./run.sh run
```

This brings up our two containers.
In a moment, you should be able to see through the logs that `marketdb` has started first, followed by `marketsvc`.

> _HINT:_ you can also use Codespaces' `Docker` extension to see which containers are running and whether they are healthy.

`marketsvc` should log a message like this once it's ready:

```sh
marketsvc-1  |  * Running on http://127.0.0.1:9090
```

Our flask app is now ready for requests.
Let's test it by running the following in a new terminal window:

```sh
curl -v http://localhost:9090/
```

You should see the heading `<h1>Welcome to MarketPlace!</h1>`.

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
> For example, the curl command above can be run by running: `./run.sh custorders`
{: .block-tip }

&nbsp;
