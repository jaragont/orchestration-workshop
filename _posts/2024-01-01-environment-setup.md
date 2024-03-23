---
title: Environment Setup
author: Aya Elsayed
category: SQLAlchemy Workshop
date: 2024-01-01
layout: post
---

In this workshop, we'll be working on `Codespaces`.

Before we begin, login to your `GitHub` account, or sign up for one if you don't already have one.

For this workshop, we'll be using the following technologies:

- [Docker](https://docs.docker.com/get-docker/)
- [Python 3.12](https://hub.docker.com/_/python)
- [Postgres](https://hub.docker.com/_/postgres)
- [SQLAlchemy](https://pypi.org/project/SQLAlchemy/)

Do not worry about installing them, we'll be installing the required tools when needed.

## Creating your Project

Throughout this workshop, we'll be using an example project `MarketPlace` (TODO: insert link).
It is a simple micro-service that manages a `postgres` database of orders, shops, customers data.
The micro-service currently uses raw SQL to manage the database, we'll be incrementally working on it to replace the raw SQL with SQLAlchemy, make use of ORMs and optimise our access patterns.

After each step, you can refer to the corresponding completed version of the code, under `solutions/step-##`.

Now, let's get started!

1. Visit (TODO: insert link) and hit the button "Use Template" > "Open in Codespaces". This will open up a copy of the project on Codespaces for you.

2. From the source control tab on the left panel of Codespaces, select "Publish Branch"

3. You'll be prompted to name your new repository. Select "public" and name your repository `<YourName>MarketPlace`

## Creating a virtual environment

To make our development experience smoother, we'll be installing the python dependencies we use in a Python virtual environment (`venv`).
Then, we'll point Codespaces to our `venv` path so that it intellisense can work and we can easily navigate to modules we use in our code.

1. Find the extensions tab in the left panel of Codespaces, and search for the `Python` extension, provided by Microsoft.
Install the extension and make sure it is enabled for your project.

2. Run the following commands to create and activate a `Python3.12` `venv`:

    ```sh
    python3.12 -m venv ./venv
    source ./venv/bin/activate
    ```

3. A prompt by the `Python` extension will pop up, asking if you'd like to use the new `./venv/bin/Python` as your current `Python` interpreter.
If not prompted, you can do this manually by clicking on the Python path in the bottom right corner of your Codespaces screen, and choosing `./venv/bin/python` for your interpreter.

4. To verify that intellisense now works correctly, open up any `.py` file in your project.
`cmd`+click on any import in your file, you should be taken to the source file of the corresponding module.
