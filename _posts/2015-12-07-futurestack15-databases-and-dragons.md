---
title: "Databases & Dragons"
date: 2015-12-07
category: talks
author: Jonathan Owens
---

Delivered at FutureStack 2015, the New Relic Developer Conference

[Watch on YouTube](https://www.youtube.com/watch?v=bV_JFrhYsqU)

At the time I was a Senior Site Reliability Engineer at New Relic, an observability company. This talk was aimed at teams running services at significant scale (50+ machines, 150+ engineers) and addresses how to choose the right data store for the right job.

## Summary

Picking a data store is a lot like rolling the dice — vendor websites are full of buzzwords but tell you little about actual fit. This talk borrows the framing of tabletop RPG character sheets to give each data store a memorable identity: stats, strengths, weaknesses, and personality.

The core thesis: **find a storage system that fits your data, not the other way around.**

### Data types and their stores

- **Event data** (the Human) — discrete records of things that happened. Fits monitoring, AdTech, analytics. New Relic Insights handled this well; open-source alternatives include Graphite, Elasticsearch, and InfluxDB.
- **Time-slice data** (the Bard) — summaries of events over a period. Stores hundreds of times more data than raw events. Cassandra is an excellent fit here; KairosDB and OpenTSDB layer time-series semantics on top of it. Graphite works at smaller volumes.
- **Relational data** (the Elf/Wizard) — financial, medical, auditing, PII. MySQL and PostgreSQL. Postgres especially: powerful, consistent, durable. If your answers to "how structured?" and "how consistent?" are "perfectly," you need Postgres somewhere.
- **Binary/blob data** (the Dwarf) — images, video, documents. S3 is the right place. Store the file in S3, store the metadata (dimensions, format, etc.) in a separate system. Don't put blobs in a SQL database.
- **Caching** (the Gnome tinkerers):
  - *Redis* — structured cache: sets, hashes, atomic counters. Scales to a single machine; great for disarming concurrency problems.
  - *Memcache* — simpler, more scalable. Pure key/value. Pair with McRouter (from Facebook) for replication and more complex topologies.

### Forming a party

In tabletop RPGs you never play alone — characters compensate for each other's weaknesses. The same applies to data stores. The Synthetics team at New Relic ran Insights (event), S3 (binary), and Postgres (relational) together: each system owned its domain, failures were isolated, and each could be scaled independently.

Rules for a good party:
1. Pick stores that compensate for each other's weaknesses.
2. Pick stores well-suited to their task — don't jam binary data into SQL.
3. Pick stores you understand. Ten years of MySQL > zero days of Postgres.

**Have services own their data.** Other services should talk to a service API, not reach into its database directly. This prevents the "Death Star architecture" where everything is connected to everything and nothing can move.

### Q&A highlights

- **Dynamo vs. Cassandra**: comes down to managed vs. self-operated — their data models are similar enough that the choice is mostly operational.
- **In-memory databases**: good fit when retention is short or failover isn't critical (finserv, AdTech). Redis covers a lot of that ground.
- **Kafka for queues**: great at scale with multiple consumers and persistent storage. Redis works fine for smaller queue needs.
- **Migrating between stores**: avoid having to do it by thinking about data type up front.
