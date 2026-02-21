---
title: "Databases & Dragons"
date: 2015-12-07
category: talks
author: Jonathan Owens
---

## Notes

Delivered at FutureStack 2015, the New Relic Developer Conference in San Francisco. This talk was aimed at teams running services at mild scale (50+ machines, 150+ engineers) and addresses how to choose the right data store for the right job.

## Video

<iframe width="560" height="315" src="https://www.youtube.com/embed/bV_JFrhYsqU" frameborder="0" allowfullscreen></iframe>


## Summary

Picking a data store is a lot like rolling the dice — vendor websites are full of buzzwords but tell you little about actual fit. This talk borrows the framing of tabletop RPG character sheets to give each data store a memorable identity based around its stats, strengths, weaknesses, and personality.

Always try to find a storage system that fits your data, not the other way around.

### Data types and their stores

- **Event data** (the Human): discrete records of things that happened. Fits monitoring, AdTech, analytics. New Relic Insights handled this well; open-source alternatives include Graphite, Elasticsearch, and InfluxDB.
- **Time-slice data** (the Bard): summaries of events over a period. Stores hundreds of times more data than raw events. Cassandra is an excellent fit here; KairosDB and OpenTSDB layer time-series semantics on top of it. Graphite works at smaller volumes.
- **Relational data** (the Elf/Wizard): financial, medical, auditing, PII. Examples: MySQL and PostgreSQL. Postgres is especially powerful, consistent, and durable. If your answers to "how structured?" and "how consistent?" are "perfectly," you need Postgres somewhere.
- **Binary/blob data** (the Dwarf): images, video, documents, compressed blocks. S3 (and its compatible competitors) is the right place. Store the file in S3, store the metadata (dimensions, format, etc.) in a separate system. Don't put blobs in a SQL database.
- **Caching** (the Gnome tinkerers): hot data structures with light durability requirements. Two market leaders:
  - *Redis*: structured cache: sets, hashes, atomic counters. Scales to a single machine; great for disarming concurrency problems.
  - *Memcache*: simpler, more scalable. Pure key/value. Pair with McRouter (from Facebook) for replication and more complex topologies.

### Forming a party

In tabletop RPGs you never play alone. By design, the characters must compensate for each other's weaknesses to have a successful experience. The same applies to data stores. The Synthetics team at New Relic ran Insights (event), S3 (binary), and Postgres (relational) together: each system owned its domain, failures were isolated, and each could be scaled independently.

Rules for a good party of datastores:

1. Pick stores that compensate for each other's weaknesses.
2. Pick stores well-suited to their task. Don't jam binary data into SQL just because you already have SQL, you'll burden your database and never scale your binaries. Choose the right tool.
3. Pick stores you understand. Ten years of MySQL > zero days of Postgres. Most of the widely used storage systems are incredibly versatile and it takes years to develop mastery of them. If you're not sure what to do, reach for what you know.

A very strong point of leverage here is to **have services own their data.** Other services should talk to a service API, not reach into its database directly. This prevents the "Death Star architecture" where everything is connected to everything and nothing can move. It also gives you freedom to choose a datastore suitable for the purpose rather than compromising among the many users of a centralized system.

### Q&A highlights

- **Dynamo vs. Cassandra**: comes down to managed vs. self-operated — their data models are similar enough that the choice is mostly operational.
- **In-memory databases**: good fit when retention is short or failover isn't critical (finserv, AdTech). Redis covers a lot of that ground.
- **Kafka for queues**: great at scale with multiple consumers and persistent storage. Redis works fine for smaller queue needs.
- **Migrating between stores**: avoid having to do it by thinking about data type up front.
