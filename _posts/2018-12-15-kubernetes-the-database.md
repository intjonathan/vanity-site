---
title: "Kubernetes the Database"
date: 2018-12-15
category: talks
author: Jonathan Owens
---

## Notes

A talk given with Maryum Styles at KubeCon + CloudNativeCon 2018 in Seattle, WA.

We were working on a team operating a Mesos cluster and needed more advanced inventory and orchestration tools than Mesos could provide. We created a Kubernetes API server cluster with custom resources, along with inventory software to populate objects based on AWS and datacenter inventory. With the broad client support for first-class operation locking and object inventory in the Kubernetes ecosystem, we could accelerate our cluster management tooling dramatically.

I don't like the title we ended up with. At the time I imagined Yogurt from Spaceballs [going through his merchandise](https://www.youtube.com/watch?v=fgRFQJCHcPw) ("Spaceballs the lunchbox, Spaceballs the flame thrower!"), sort of as a spin on how Kubernetes gets used for everything ("Kubernetes the webserver! Kubernetes the database!"). The pattern we're  describing is using the Kubernetes API server with custom resources as an inventory server for cloud and physical machines, along with describing and orchestrating cluster state. Perhaps a better title would be "Kubernetes Inventory for Non-K8s Resources".

## Video

<iframe width="560" height="315" src="https://www.youtube.com/embed/eja7b3tahMg" frameborder="0" allowfullscreen></iframe>

## Abstract

> In the operations world, one of the hardest problems is keeping track of your inventory: Which machines belong to which teams? Which machines are in service? How long have they been there? At New Relic, the ability to keep track of a massive inventory that runs across multiple providers quickly became an unbearable task so much so that it required designing a completely new central tracking system that could scale with a large infrastructure. In this talk, you'll learn how Jonathan Owens and Maryum Styles used the Kubernetes API server to jump-start this design and create a unified infrastructure description service. They will share how they defined resources, created controller services, and dramatically decreased the process of manual updates.

## Summary

On using Kubernetes API components as a database for host inventory:

- **The inventory challenge at New Relic**
  - Container Fabric team running ~1,000 machines on DC/OS (Mesos) across 2 regions and 3 infrastructure providers including bare-metal. Managed with Ansible and Terraform. Too many servers and clusters to manage with Ansible alone, and no "rolling restart" primitive without centralized data.
- **Alternatives considered and rejected**
  - Terraform state: only updates on demand, clumsy file-based access
  - Commercial DCIM tools (Device42, Nlyte): focused on physical rack-level details, no support for higher-level abstractions like clusters
  - DC/OS itself: reactive/responsive model with no declarative state
- **Requirements for a solution**
  - Single source of truth, updatable from anywhere without VPN tunneling
  - Compatible with existing Ansible inventory
  - Production quality with support for higher-level abstractions (clusters, roles, log hosts)
- **The Kubernetes API server as the answer**
  - Provides centralized declarative configuration, extensible via Custom Resource Definitions (CRDs)
  - Can be run standalone -- "Kubernetes the easy way" -- just the API server, etcd, and controller manager, no kubelets, no CNI, no pod networking
- **System architecture: three components**
  - CRD definitions for host and cluster objects with labels for filtering and annotations for metadata
  - Fetchers: lightweight controllers (one per provider -- AWS, SoftLayer, datacenter) that query provider APIs and push host objects into the API server using patch operations
  - An Ansible inventory adapter: a Python script querying the API server and outputting JSON so Ansible could use it as an inventory for any cluster.
- **Expanding the pattern beyond inventory**
  - Optimistic concurrency enables safe multi-client coordination
  - Monitoring: replaced git-repo-polling workflow to populate monitoring configuration with dynamic reconfiguration via API server watches.
  - Early steps toward self-managed clusters with on-host agents requesting Ansible runs

**Final Thoughts**: Kubernetes is a hamburger made of many parts, but the cheese (the API server and its declarative state model) is delicious on its own -- look at the opinions about distributed systems baked into these components and apply them to your own infrastructure
