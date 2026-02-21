---
title: "The Monolith Destruction Toolkit"
date: 2016-12-07
category: talks
author: Jonathan Owens
---

## Notes

A talk given with Jose Fernandez at FutureStack, the New Relic Developer Conference, on 7 December 2016 in San Francisco.

We presented service architecture patterns our team used to incrementally decompose a monolithic Java service at New Relic. The "collector" processed hundreds of millions of time series data points per minute and had grown alongside the company until nobody fully understood its behavior. Rather than a risky full rewrite, we broke it apart one data stream at a time.

## Video

<iframe width="560" height="315" src="https://www.youtube.com/embed/zjZsZvZr7QY" frameborder="0" allowfullscreen></iframe>

## Summary

### The monolith problem

New Relic's collector service was a monolithic JVM that ingested, aggregated, and stored all customer telemetry data. It had grown organically with the company's superlinear traffic growth until the codebase was too large for anyone to fully understand. Every engineer who touched it "caught the sad." It couldn't be rewritten all at once, though, because the business depended on it running continuously. The approach had to be incremental: break it apart one chunk at a time.

### Choosing what to extract first

More services isn't the goal; reducing pain is. Developer pain was tempting to target, but scale pain turned out to be the right angle. Time series data ("time slices") was the biggest scale concern: hundreds of millions per minute, requiring both aggregation and storage to MySQL. Time slices also had a useful property: isolation. Only one system wrote them, one stored them, and one queried them, making them a clean extraction target.

### Building in the dark with Kafka

The mandate was zero customer-facing impact. The team introduced Kafka as a data bus, cloning time slice data out of the collector into a parallel "dark" pipeline. This was a small change to the collector, and once the data was flowing into Kafka, the monolith didn't need to be touched again. The dark pipeline stored data to Cassandra instead of MySQL, giving the team freedom to iterate without affecting customers.

### Shadow reads for verification

To prove the new pipeline was correct, the team added instrumentation to the existing query service. When a query came in, results and performance data from the legacy MySQL path were recorded alongside the same query executed against Cassandra. Both sets of results were sent to New Relic Insights, where the team could chart a "horse race" between the two systems. Cassandra was significantly faster, and the results matched, giving the team and leadership confidence that the new pipeline was sound.

### Splitting services further

The initial extraction produced a single "minute aggregator" service that consumed data from Kafka, aggregated it, and wrote to Cassandra. It didn't perform well enough. Since the team was working in the dark, they could safely experiment: splitting aggregation (CPU and memory intensive) from writing (I/O and network intensive) into separate services with separate performance profiles. Half the team was skeptical, but the data proved it out. Each finer split also exposed more data into Kafka, unlocking new opportunities. The aggregated data queue, for instance, turned out to be the "data firehose" that other teams had always wanted access to.

### Finding service boundaries

Two rules guided decomposition:

1. Conway's Law: service boundaries should match organizational communication boundaries. One team owned the monolith and steered which pieces got extracted and by whom.
2. Experimental proof: within a team, further splits had to be justified by data, whether that was performance improvement, operational simplicity, or enabling team growth.

The connecting tissue was Kafka. When services communicate through a shared stream processor rather than bespoke APIs, they plug together in a more uniform way.

### Putting ops in the dev

The collector had burned out engineers, partly because the team was all developers with no operations perspective. Through New Relic's "Upscale" reorganization, SREs joined product teams permanently rather than being loaned out on a ticket queue. The team went from waiting weeks for infrastructure changes to pairing in a room and finishing in a day. SREs brought a broader perspective to design decisions, and teams with a permanent reliability engineer shipped faster and made fewer architectural mistakes. Jonathan and Jose proposed calling this role a "Product Reliability Engineer" to distinguish it from the traditional loaned-out SRE model.

### Containers as architecture guardrails

Docker was central to the new pipeline. Beyond packaging and deployment convenience, Docker is opinionated about software architecture in ways that encourage better practices. It surfaces operational concerns to developers, provides standard interfaces for restarts, logging, deployment, and shells, and ensures that anyone joining the team learns one way to operate everything. The team even dockerized Cassandra, spending six to eight weeks adapting their deployment tool (Centurion) to handle persistent disks and JMX interactions. The payoff was going from hours-long, multi-team database deploys to thirty-minute, single-team deploys, a benefit that compounded over the following year.

### The shopping list

1. Treat data intake as a stream and put it in a stream processor
2. Break up the monolith one stream at a time
3. Put ops in your dev
4. Do it in containers
