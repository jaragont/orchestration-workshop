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

After each part, we can refer to the corresponding completed version of the code, under the branch `part_#`.

**This is quite important if you fall behind, as you can start from the last complete milestone.**

Now, let's get started!

1. Visit the [tutorial repo](https://github.com/jaragont/orchestration-workshop-tutorial) and **fork** the repository by clicking "Fork" > "Create a new fork"
You will be redirected to a fork of this repository in your account or organization.

1. **Make sure to uncheck the box `Copy the 'main' branch only`**, to include all the branches.

2. Once our repo is created, click on the `Code` menu, and from the `Codespaces` tab, click `Create codespace on main`.
This will open the code in GitHub `Codespaces` where we will be incrementally making updates to the code.


### Repository Structure

[This](https://github.com/jaragont/orchestration-workshop-tutorial) repository contains all the information required for the workshop.

- `/data`: data files
- `/workshop-project/basic`: basic solution of the proposed part
- `/workshop-project/advanced`: advanced solution of the proposed part
- `/requirements.txt`: Python dependencies

---

## Creating a virtual environment

We'll be installing our Python dependencies in a Python virtual environment (`venv`).

Even while on `Codespaces`, it is good practice to create a `venv` for our project's dependencies.

To create and activate a `Python3.12` `venv`:

```sh
python3.12 -m venv /workspaces/orchestration-workshop-tutorial/.venv
source /workspaces/orchestration-workshop-tutorial/.venv/bin/activate
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

> ⚠️ This `venv` contains all your base dependencies, ensure you have not deactivated it accidentally when switching branches.

---

## Running Locally

If we prefer to work outside `Codespaces`, you can clone your fork first:

```sh
git clone <our-fork-url>
```

The examples reference paths relative to the `Codespaces` base location; ensure you modify them as appropriate for your environment.

---