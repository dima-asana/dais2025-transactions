Content for [DAIS 2025 talk on transaction conflicts in Databricks](https://www.databricks.com/dataaisummit/session/reducing-transaction-conflicts-databricks-fundamentals-and-applications).

[Slides](https://docs.google.com/presentation/d/1wFMKUhpWEBCRbQjWgupUqO2otELlfaSU)

## Behavior for concurrent transactions

In the subfolders numbered 1-12, you can look at what happens when two
specific transactions attempt to run at the same time on the Databricks
platform.
Each folder contains
- a main.ipynb which contains the table setup and analysis
- 1.ipynb and 2.ipynb for the content of the two transactions.
- A link in the README.md for a recording of two transactions run in parallel

These results were obtained on Databricks 16.4 LTS runtime running on a
personal compute cluster in dedicated mode.

## Why we worked on this problem

Asana uses a [Work Graph data model](https://asana.com/resources/work-graph)
to provide flexible performant functionality for many types of data models.

Sometimes, we want to look at functions common to many different data models -- e.g. what the last modification type was
on one of many data models.

Our analytical processing is substantially powered by Databricks.
There is a separate Delta Lake table for each of ~700 work graph models.

To trigger parallel incremental computation, we started listening to the
change feed on each data model table, with some MERGE INTO operations
running on changes to many of these tables.

Some operations could have ~700 concurrent triggers.
We ran into many concurrent transaction conflicts with this setup.

## Our solution evolution

At first, we solved this problem through partitioning
(similar to subfolder 7).  We had an intermediate partitioning stage
partitioned by what upstream table was writing to it.

This worked well for writes.  A year later we wanted to take advantage of
[liquid clustering](https://docs.databricks.com/aws/en/delta/clustering)
for faster reads.  Liquid clustering is not compatible with partitioning
because it requires relinquishing control over file splits of data.
We implemented an intermediate stage `INSERT`-based blind append (similar
to subfolder 1) -- a table into which we write the change feeds from all
the tables we care about, and then triggering `MERGE INTO` processing on
change feeds on just that one table.


