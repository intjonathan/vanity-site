---
title: "Kubernetes the Database"
date: 2018-12-15
category: talks
author: Jonathan Owens
---

## Notes

A talk given with Maryum Styles at KubeCon + CloudNativeCon 2018 in Seattle, WA.

We were working on a team operating a Mesos cluster and needed more advanced inventory and orchestration tools than Mesos could provide. We created a Kubernetes API server cluster with custom resources, along with inventory software to populate objects based on AWS and datacenter inventory. With the broad client support for first-class operation locking and object inventory in the Kubernetes ecosystem, we could accelerate our cluster management tooling dramatically.

## Abstract

> In the operations world, one of the hardest problems is keeping track of your inventory: Which machines belong to which teams? Which machines are in service? How long have they been there? At New Relic, the ability to keep track of a massive inventory that runs across multiple providers quickly became an unbearable task so much so that it required designing a completely new central tracking system that could scale with a large infrastructure. In this talk, you'll learn how Jonathan Owens and Maryum Styles used the Kubernetes API server to jump-start this design and create a unified infrastructure description service. They will share how they defined resources, created controller services, and dramatically decreased the process of manual updates.

## Video

<iframe width="560" height="315" src="https://www.youtube.com/embed/eja7b3tahMg" frameborder="0" allowfullscreen></iframe>
