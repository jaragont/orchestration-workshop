---
layout: home
title: "Orchestrating Data Pipelines in Python: From Generation to Quality"

permalink: /
---

## Welcome to the Orchestrating Data Pipelines Workshop

### Meet María

María is a data analyst at a UN-affiliated organization focused on renewable energy. María’s mission: conduct an exploratory data analysis (EDA) to compare renewable energy adoption between the Iberia region and North American countries. This analysis is vital for identifying trends, informing policy, and assessing progress in renewable initiatives.

![Illustration of María, a data analyst at a UN-affiliated organization](assets/maria-image.jpg)
*Source of generated image on this page: Google Gemini*

### The challenge

However, María faces a challenge. Her analysis requires comparing Iberia vs. North America regions, but she doesn't have this regional data ready-made. She only has country-level datasets—population, energy consumption, and renewable generation—along with a custom taxonomy defining regional groupings. To perform her analysis, she needs to transform this raw country data into regional insights by combining datasets, normalizing metrics per capita, and aggregating to regional levels. Doing this manually is complex, error-prone, and inefficient, especially when tracking changes over time.

### The adventure

This workshop is our call to adventure: we show how to build a structured, reliable ETL pipeline using Python tools. We learn to automate data preparation and analysis with [pandas](https://pandas.pydata.org/), [Dagster](https://dagster.io/), [Pandera](https://pandera.readthedocs.io/en/stable/#), and Soda—transforming a manual scramble into a predictable, trustworthy process.

For each ETL stage (Extract, Transform, Load), we first use plain Python and pandas, then elevate the workflow with specialized orchestration tools. This "before and after" approach highlights the benefits of advanced, scalable data pipelines.

By the end, we are equipped to confidently compare renewable adoption across regions—just like María!

---

### Who is this workshop for?

This workshop is designed for data analysts, engineers, students, and anyone interested in automating data workflows using Python. Basic familiarity with Python and pandas is recommended.

### What will we learn?

- How to automate data extraction, transformation, and loading (ETL)
- How to validate data quality and enforce contracts
- How to orchestrate scalable, maintainable data pipelines

### Workshop Structure

This workshop is organized into four main parts, each focusing on a critical stage of the data pipeline:

1. **Extract (E)** - Data ingestion and collection
2. **Transform (T)** - Data cleaning and processing
3. **Load (L)** - Data storage and output
4. **Quality Control** - Data validation and monitoring

**Learning Approach:** For each part, we follow a progressive methodology:

- Start with basic implementation using plain Python and pandas
- Introduce advanced orchestration tools to demonstrate clear benefits
- Provide hands-on exercises where participants implement the advanced solutions
- Speakers will be available for guidance and support during practice sessions

This structure ensures we understand both the fundamentals and the power of modern data pipeline tools.

---

Ready to get started? Jump into the first module and begin our data pipeline adventure!
