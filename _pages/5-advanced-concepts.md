---
title: "Part 5: Advanced Orchestration Concepts"
author: Daniel Ortiz, Juan Aragon
category: Orchestrating Data Pipelines Workshop
date: 2025-09-26
layout: post
---

## Scene: María Looks Beyond the Basics

With a working ETL pipeline and quality control in place, María is ready to take her orchestration skills to the next level. In production, there are more advanced concepts and tools to master. In this section, we introduce those topics, with examples and links to official documentation for further exploration.

![Illustration of María, a data analyst at a UN-affiliated organization](../../assets/maria-part-5.png)

---

### What We Will Learn

In this section, we will briefly introduce important concepts to:

- Automate and schedule our pipelines using `Dagster`
- Deploy `Dagster` in production environments
- Manage data transfer efficiently with `IOManager`
- Trigger pipelines reactively with sensors

---

## 1. Scheduling & Job Management

Imagine María needs to produce a version of this report every month.

In production, pipelines should run automatically, not just on demand. Dagster supports robust scheduling and job management, allowing us to automate asset materializations and group related tasks.

**Example: Defining a Schedule**
```python
from dagster import ScheduleDefinition, define_asset_job

my_job = define_asset_job("my_job")

my_schedule = ScheduleDefinition(
    job=my_job,
    cron_schedule="0 8 * * *",  # Every day at 8am
)
```

- [Dagster Scheduling Documentation](https://docs.dagster.io/concepts/partitions-schedules-sensors/schedules)
- [Job and Asset Grouping](https://docs.dagster.io/guides/build/jobs)

---

## 2. Production Deployment & Infrastructure

The default `Dagster` development setup (`dg dev`) is not suitable for production. For reliability and scalability, we need to:
- Use a persistent database (e.g., `PostgreSQL`) for `Dagster` metadata
- Deploy `Dagster` with `Docker`, `Kubernetes`, or other orchestration tools across multiple nodes
- Configure distributed execution for large workload

> In the `dg dev` setup, the default metastore is a local `sqlite`.

The official documention shows you how to implement this transition according to your specific architecture requirements.

**Resources:**
- [Dagster Deployment Overview](https://docs.dagster.io/deployment)
- [Productionizing Dagster](https://docs.dagster.io/guides/operate/dev-to-prod)

---

## 3. IOManager for Efficient Storage

IO Managers control how data is passed between assets. The default is local filesystem storage, but for real pipelines, we may want to use cloud storage (`S3`, `GCS`, etc.) or databases.

This is quite important a distributed context, as different components of the pipeline may not share a filesystem.

> In the `dg dev` setup, the default store is a local filesystem IOManager that leverages `pickle`.

**Example: Using an S3 IOManager**

This manager will store the intermediate objects in an `S3` bucket, instead of your local file system.

```python
from dagster_aws.s3 import s3_pickle_io_manager

def my_assets():
    ...

# In our Definitions:
defs = Definitions(
    assets=[my_assets],
    resources={"io_manager": s3_pickle_io_manager},
)
```

- [Dagster IO Managers](https://docs.dagster.io/concepts/io-management/io-managers)

---

## 4. Event-Driven Pipelines with Sensors (and More)

Beyond schedules, `Dagster` supports sensors—reactive triggers that launch jobs in response to external events (like new files, API calls, etc.).

**Example: File Sensor**
```python
# Sensor will be evaluated at least every 30 seconds
@dg.sensor(job=my_job, minimum_interval_seconds=30)
def new_file_sensor():
  ...
```

- [Dagster Sensors](https://docs.dagster.io/concepts/partitions-schedules-sensors/sensors)

**Explore Further:**
- Complex event-driven architectures
- Multi-asset sensors
- Custom triggers and integrations

---

## Next Steps & Resources

- [Dagster Tutorials](https://docs.dagster.io/tutorial)
- [Dagster Community](https://dagster.io/community)
- [Advanced Guides](https://docs.dagster.io/guides)
