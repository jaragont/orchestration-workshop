---
title: "Environment Setup"
author: Daniel Ortiz, Juan Aragon
category: Orchestrating Data Pipelines Workshop
date: 2024-01-01
layout: post
---

## Prerequisites

- GitHub account
- Basic familiarity with Python and Git
- Access to GitHub Codespaces (or ability to run Python locally)
- Python 3.12 installed (for local development)

---

## Creating our Project

Throughout this workshop, we'll be using an [example project to help María in her task](https://github.com/jaragont/orchestration-workshop-tutorial).

After each part, we can refer to the corresponding completed version of the code, under the branch `part-#`.

Now, let's get started!

1. Visit the [tutorial repo](https://github.com/jaragont/orchestration-workshop-tutorial) and **fork** the repository by clicking "Fork" > "Create a new fork"
You will be redirected to a fork of this repository in your account or organization.

1. **Make sure to uncheck the box `Copy the 'main' branch only`**, to include all the branches.

2. Once our repo is created, click on the `Code` menu, and from the `Codespaces` tab, click `Create codespace on main`.
This will open the code in GitHub `Codespaces` where we will be incrementally making updates to the code.

---

### Repository Structure

[This](https://github.com/jaragont/orchestration-workshop-tutorial) repository contains all the information required for the workshop.

- `/src`: Source code for the ETL pipeline
- `/workshop-project/basic`: basic solution of the proposed part
- `/workshop-project/advanced`: advanced solution of the proposed part
- `/requirements.txt`: Python dependencies

---

## Running Locally

If we prefer to work outside Codespaces:

```sh
git clone <our-fork-url>
cd workshop-repo
python3.12 -m venv ./venv
source ./venv/bin/activate
pip install -r requirements.txt
```

---

## Creating a virtual environment

We'll be installing our Python dependencies in a Python virtual environment (`venv`).

While on `Codespaces`, creating a virtual environment for our project doesn't add much value, if we were developing locally or on a host shared by multiple projects, it is good practice to create a `venv` for our project's dependencies.

To create and activate a `Python3.12` `venv`:

```sh
python3.12 -m venv ./venv
source ./venv/bin/activate
```

> _NOTE_: You can exit out of the `venv` by typing:
>
> ```sh
> deactivate
> ```

Once we've activated our `venv`, we can now install `PyPI` dependencies.
We'll be installing a few throughout this tutorial.

### Install Dependencies

```sh
pip install -r requirements.txt
```

---
