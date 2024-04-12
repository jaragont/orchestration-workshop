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

- [Docker](https://docs.docker.com/get-docker/)
- [Python 3.12](https://hub.docker.com/_/python)
- [Postgres](https://hub.docker.com/_/postgres)
- [SQLAlchemy](https://pypi.org/project/SQLAlchemy/)
- [Flask](https://flask.palletsprojects.com/en/3.0.x/)

Do not worry about installing them, we'll be installing the required tools when needed.

## Creating your Project

Throughout this workshop, we'll be using an [example project `Marketplace`](https://github.com/rhythm-patel/sqlalchemy-workshop).
It is a simple micro-service that manages a `postgres` database of orders and customers.
The micro-service currently uses raw SQL to manage the database, we'll be incrementally working on it to replace the raw SQL with SQLAlchemy, make use of ORMs and optimise our access patterns.

After each step, you can refer to the corresponding completed version of the code, under the branch `step-#-<step-title>`.

Now, let's get started!

1. Visit the [example repo](https://github.com/rhythm-patel/sqlalchemy-workshop) and hit the button "Use Template" > "Create new repository".
You will be redirected to a page to create a new repository in your org based on our example repo.

    ![image1](/sqlalchemy-wkshop/assets/gitbook/images/use_template.png)

2. Make sure to tick the box `include all branches`, then give your repo a name.

    ![image2](/sqlalchemy-wkshop/assets/gitbook/images/create_repo.png)

3. Once your repo is created, click on the `Code` menu, and from the `Codespaces` tab, click `Create codespace` on main.
This will open the code in a `Codespaces` container where we will be incrementally making updates to the code.

    ![image3](/sqlalchemy-wkshop/assets/gitbook/images/codespaces.png)

## Useful Extensions

In this tutorial, two useful VS Code extensions will make your development experience smoother:

1. The [Python extension](https://marketplace.visualstudio.com/items?itemName=ms-python.python)

2. The [Docker extension](https://code.visualstudio.com/docs/containers/overview#_installation)

Find the Extensions tab in the left panel of `Codespaces`, and for each extension, search for it by name. Then hit the install button.
Both extensions are provided by Microsoft.

## Creating a virtual environment

To get [intelliSense](https://code.visualstudio.com/docs/editor/intellisense) working, we'll be installing the Python dependencies we use in a Python virtual environment (`venv`).
Then, we'll point the `Python` extension to our `venv` Python path so that `Codespaces` can recognise the modules we use in our code.

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
