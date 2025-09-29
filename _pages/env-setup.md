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

## Repository Structure

The repository contains (**TODO:** refine this section):

- `/src`: Source code for the ETL pipeline
- `/data`: datasets
- `/requirements.txt`: Python dependencies

---

## Working with Branches

Each step of the workshop is available in a branch named `step-#-<step-title>`.
To switch branches locally, use:

```sh
git checkout step-1-setup
```

---

## Creating our Project

Throughout this workshop, we'll be using an [example project to help María in her `EDA` task](https://bbgithub.dev.bloomberg.com/dortizcosta/workshop-repo).

After each step, we can refer to the corresponding completed version of the code, under the branch `step-#-<step-title>`.

Now, let's get started!

1. Visit the [example repo](https://bbgithub.dev.bloomberg.com/dortizcosta/workshop-repo) and **fork** the repository by clicking "Fork" > "Create a new fork".
You will be redirected to a fork of this repository in your account or organization.

2. **Make sure to uncheck the box `Copy the 'main' branch only`**, to include all the branches.

3. Once our repo is created, click on the `Code` menu, and from the `Codespaces` tab, click `Create codespace on main`.
This will open the code in GitHub `Codespaces` where we will be incrementally making updates to the code.

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

## Recommended Extensions

To make our development experience smoother, we want to enable [IntelliSense](https://code.visualstudio.com/docs/editor/intellisense). To do that for `Python`, we'd need to install the official VSCode [Python extension](https://marketplace.visualstudio.com/items?itemName=ms-python.python).

Other useful extensions:
- [GitLens](https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens) for Git insights

Find the Extensions tab in the left panel of `Codespaces`, and search `Python`.
From the results list, find the extension named `Python` provided by Microsoft.
Hit install.

In the previous step, we've created a `venv` where we will be installing some extra dependencies. To ensure that `IntelliSense` will be able to recognise these dependencies, we need to point the `Python` extension to our `venv`'s Python path.

To do that, once the extension has been installed, open up any python file, and on the bottom right corner of your `Codespaces` screen, you should see that `Codespaces` has detected the language to be `Python` and it shows the `Python` path of the interpreter it is currently using.

Click on the path to change it.

A drop down menu should appear.
Select `Python3.12`, the version with the path `./venv/bin/python`.
