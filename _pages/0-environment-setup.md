---
title: Environment Setup
author: Aya Elsayed, Rhythm Patel
category: SQLAlchemy Workshop
date: 2024-01-01
layout: post
---

In this workshop, we'll be working on `GitHub Codespaces`.

Before we begin, log in to your `GitHub` account, or sign up for one if you don't already have one.

For this workshop, we'll be using the following technologies:

- [Python 3.12](https://hub.docker.com/_/python)
- [SQLite](https://www.sqlite.org/index.html)
- [SQLAlchemy](https://pypi.org/project/SQLAlchemy/)
- [FastAPI](https://fastapi.tiangolo.com/)

Do not worry about installing them, we'll be installing the required tools when needed.

## Creating your Project

Throughout this workshop, we'll be using an [example project `Marketplace`](https://github.com/rhythm-patel/sqlalchemy-workshop).
It is a simple microservice that manages a `SQLite` database of orders and customers.
The microservice currently uses raw SQL to manage the database, we'll be incrementally working on it to replace the raw SQL with SQLAlchemy, make use of ORMs, and then optimise our access patterns.

After each step, you can refer to the corresponding completed version of the code, under the branch `step-#-<step-title>`.

Now, let's get started!

1. Visit the [example repo](https://github.com/rhythm-patel/sqlalchemy-workshop) and hit the button "Use Template" > "Create new repository".
You will be redirected to a page to create a new repository in your org based on our example repo.

    ![image1](/sqlalchemy-wkshop/assets/gitbook/images/use_template.png)

2. **Make sure to tick the box `include all branches`**, then give your repo a name.

    ![image2](/sqlalchemy-wkshop/assets/gitbook/images/create_repo.png)

3. Once your repo is created, click on the `Code` menu, and from the `Codespaces` tab, click `Create codespace on main`.
This will open the code in GitHub `Codespaces` where we will be incrementally making updates to the code.

    ![image3](/sqlalchemy-wkshop/assets/gitbook/images/codespaces.png)

## Useful Extensions

In this tutorial, the [Python extension](https://marketplace.visualstudio.com/items?itemName=ms-python.python) for VSCode will make your development experience smoother.

Find the Extensions tab in the left panel of `Codespaces`, and search `Python`.
The extension is provided by Microsoft.
Hit install.

## Creating a virtual environment

We'll be installing our Python dependencies in a Python virtual environment (`venv`).

While on `Codespaces`, creating a virtual environment for our project doesn't add much value, if you were developing locally or on a host shared by multiple projects, it is good practice to create a `venv` for your project's dependencies.

Once we have our `venv` setup, we'll point the `Python` extension to our `venv` Python path so that `Codespaces` can recognise the packages we use in our code, and [IntelliSense](https://code.visualstudio.com/docs/editor/intellisense) will work correctly.

1. In the previous section, you should have installed the `Python` extension.
If you haven't done so already, please do that now.

2. Run the following commands to create and activate a `Python3.12` `venv`:

    ```sh
    python3.12 -m venv ./venv
    source ./venv/bin/activate
    ```

    > _NOTE_: You can exit out of the `venv` by typing:
    >
    > ```sh
    > deactivate
    > ```

3. A prompt by the `Python` extension will pop up, asking if you'd like to use the new `./venv/bin/Python` as your current `Python` interpreter.
If not prompted, you can do this manually by clicking on the Python path in the bottom right corner of your `Codespaces` screen, and choosing `./venv/bin/python` for your interpreter.

&nbsp;
