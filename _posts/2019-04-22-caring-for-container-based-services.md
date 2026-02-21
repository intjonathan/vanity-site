---
title: "Caring for Container-Based Services with Checks, Monitoring, and Alerts"
date: 2019-04-22
author: Jonathan Owens
---

_I originally wrote this as internal documentation for New Relic engineers using an internal container platform, then adapted and published on the New Relic marketing blog. See an [archival copy](https://web.archive.org/web/20210928093648/https://newrelic.com/blog/how-to-relic/container-service-checks) on the wayback machine._

New Relic's container orchestration team—known internally as Container Fabric (CF)—builds and maintains the company's platform for deploying and maintaining containerized services. Our developers build releases as [Docker](https://www.docker.com/) images, and we run those as containers on our platform.

The environment we manage isn't massive by modern enterprise standards, but it's large and growing rapidly. We currently manage some 1,000 machines, most of which run on physical hardware spread across three infrastructure providers in multiple data centers.

For teams running services on the CF platform, we believe the reliability of those services exists on a spectrum:

![Service reliability spectrum](/assets/images/container-services/monitoring-services-spectrum.jpg)

On the left side of the spectrum, health and readiness checks help us ensure that services are running and ready when they're deployed. In the middle, monitoring and alerting solutions help us keep our platform observable—and our operators happy and rested. On the right end, happy customers and well-rested operators are the result of development and operations discipline that produces resilient services.

But what makes for effective health checks and monitoring for container-based services? How can we make sure our services really are running, ready, and observable every time? To answer that question, the CF team has evolved a set of best practices for maintaining the integrity of services running on our container platform.

## Check: Is the service running?

Health checks are critical to make sure an orchestration platform is correctly deploying and managing services at scale, so our developers define a health check for any service they run in a container. To write quality health checks, we consider five areas:

### 1. Fidelity

A health check's response code should represent the true state of the service's health. A simple "can I respond at all" check is rarely sufficient for a service of any complexity. For example, if a service fails its health check in CF, it'll be killed and pulled from the load balancer and then restarted.

Typically, if a service fails a health check because its dependencies have failed, it'll start "flapping"—repeatedly restarting until its dependencies are running. From an orchestration perspective, this is generally OK, so long as the service records (in a log line or traced error, for example) why it restarted, so operators or developers can discover and resolve the dependency failure. Still, we advise our developers to include some error handling, as a "flapping" instance can delay functionality for any other service instances that require it to be running.

### 2. Latency

A service's health check should respond with a status as quickly as possible. A health check endpoint will be hit many millions of times over the lifetime of a service, so it needs to be fast. For this reason, we advise developers not to send CSS or too much structured data in a check's response. Further, development teams should consider also how much (if any) logging or tracing they want to perform on a check's endpoint, as performance data can become cluttered or skewed by a fast endpoint with a high hit rate.

### 3. Context and status for partial failures

To achieve low latency on a health check's endpoint, you should exclude from the response additional details about a service's health. Instead, split that additional information into a separate endpoint (for example, `/status/info`) to expose failing upstream dependencies or to inspect the live service's configuration. If developers include an informational endpoint, it should reply with the same status code as a regular health check would (for example, a 200 response for healthy, and a 503 response for unhealthy), but it can reply with a richer body in all cases.

### 4. Dependency handling

Our developers carefully consider which upstream services can fail before their service becomes truly unhealthy. If possible, a service should attempt to stay running if any (or even all) of its upstream dependencies fail. A well-hardened service can continue to check in as healthy in the face of upstream service failures while presenting a partially degraded experience, working slower than normal, or pausing operation until all dependencies are available. Restarting a service because its dependencies are down rarely helps. Developers can, on the other hand, use an informational endpoint to identify failing dependencies for later troubleshooting. (Note the contrast between this and the Fidelity consideration described above.)

### 5. Cold starts and thundering herds

For HTTP services in CF, a container's networking path is used for both customer traffic and health checks. If a container's process runs out of connections, receives too many requests, or is otherwise out of capacity to service HTTP requests, its health checks will fail and the service will restart. This is not ideal, so it's something developers need to keep in mind.

## Check: Is the service ready?

For services deployed in CF, developers can define a readiness check, which provides a signal to that a deployment is safe to continue. This is a useful check if a service performs any bookkeeping during startup (for example, migrating a database, establishing partition ownership, or warming caches).

To put the health/readiness/traffic check differences in context, this chart shows how different platforms we use deal with these checks:

| System | Service running? | Service ready? | Traffic ready? |
|--------|-----------------|----------------|----------------|
| [Monit](https://mmonit.com/monit/) | [Monit checks](https://mmonit.com/monit/documentation/monit.html#Service-checks) | [Capistrano](https://capistranorb.com/) | [F5 checks](https://support.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/ltm-monitors-reference-11-2-0/1.html) |
| [DC/OS](https://dcos.io/) | [Health check](https://docs.mesosphere.com/1.10/deploying-services/creating-services/health-checks/) | [Readiness check](https://mesosphere.github.io/marathon/docs/readiness-checks.html) | Health check |
| [Kubernetes](https://kubernetes.io/) | [Liveness probe](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/) | [Readiness probe](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/) | Readiness probe |

A further complication for DC/OS is that readiness checks are executed only during deployment, and aren't evaluated during normal container restarts (for example, if the container restarted due to maintenance or failed a health check). On DC/OS, services should not rely on readiness checks as part of a healthy container startup, only as a deployment primitive.

In DC/OS, for example, the health_check setting is used for both process-running and traffic-ready checks, so a state of "live process, no traffic" can't be expressed for HTTP services. In this situation, we advise that an HTTP service shouldn't pass its health check until it's ready to handle customer traffic. After all, "ready" can take many meanings: A service may be "ready" to present a "We're sorry" page, but that doesn't mean it's available to accept traffic.

### A note on active external checks

Every HTTP service should have an active, externally initiated check that accesses the service through the load balancer endpoint, typically using [New Relic Synthetics](https://newrelic.com/products/synthetics). [New Relic APM](https://newrelic.com/products/application-monitoring) or [New Relic Insights](https://newrelic.com/products/insights) data doesn't tell you when requests fails to even reach your service, but a Synthetics check can travel the full path from the load balancer through the traffic proxy to the container and verify that the network path is working appropriately.

## Check: Is the service observable?

At New Relic, we know that monitoring an application helps keep it reliable. This applies to orchestration platforms as well, and CF exposes plenty of data about the containers it runs, primarily through Insights events. Good monitoring keeps our platform operators happy.

The CF platform runs a data gathering and aggregation stack the team calls the CF metrics pipeline. For every container on the platform, we collect container events, container metrics, and container logs.

_Note: For an explanation of Docker events see [Docker Events Explained](https://gliderlabs.com/devlog/2015/docker-events-explained/)._

We poll container metrics and send them to Insights as a number of different [event types](https://docs.newrelic.com/docs/insights/use-insights-ui/getting-started/introduction-new-relic-insights#event-type):

- `cf_docker_container_blkio`
- `cf_docker_container_cpu`
- `cf_docker_container_memory`
- `cf_docker_container_net`
- `cf_docker_event`

For example, here's a look at an Insights dashboard tracking CPU usage for a service called docker-state:

![Container Fabric Insights dashboard tracking CPU usage](/assets/images/container-services/cf-cpu-dashboard.jpg)

The events we collect power some key alerts, defined by [NRQL alert conditions](https://docs.newrelic.com/docs/alerts/new-relic-alerts/defining-conditions/create-alert-conditions-nrql-queries), that all services should use in production.

For example, here's the query we use for the CPU core count table, shown in the CPU dashboard:

```sql
SELECT average(cpu_usage_core_count) as 'CPUs' FROM cf_docker_container_cpu WHERE service_name = 'docker-state' AND environment = 'production'
```

We use this same query to set our alert condition in [New Relic Alerts](https://newrelic.com/alerts). This alert will trigger if the docker-state service uses more than 0.45 CPU cores over two minutes. In production, this service has a quota of 0.5 CPU cores, so if an operator receives this alert, they know this service is at risk of CPU throttling.

## Three key alerts to keep operators happy

Along with CPU ceilings, we've identified three more critical events in which services could disrupt services in production: out of memory (OOM) kills, SIGKILLs, and load-balancer connection limits. For each of these events, you can create an alert condition [in the Alerts UI](https://docs.newrelic.com/docs/alerts/new-relic-alerts/defining-conditions/define-alert-conditions) or with the [Alerts REST API](https://docs.newrelic.com/docs/alerts/rest-api-alerts/new-relic-alerts-rest-api/rest-api-calls-new-relic-alerts).

### 1. OOM events

An out of memory (OOM) event occurs if the processes in a container exceed their memory quota and are killed by the OOM killer. An OOM event doesn't *always* affect service availability, as a container restart can often clear the memory usage enough for it to continue. But any number of OOM kills is cause for concern. Consider setting a low-priority alert at a [warning threshold](https://docs.newrelic.com/docs/alerts/new-relic-alerts/defining-conditions/set-thresholds-alert-condition#threshold-definition) (more than one in five minutes), and a high-priority alert at the critical threshold (more than three in five minutes). In most cases, if the number of OOM events in a short (roughly five-minute) period is greater than or equal to a service's instance count, it's likely that service is down.

### 2. SIGKILL events

A SIGKILL is signal that tells a process to terminate immediately. SIGKILLs are generated if the container does not exit after receiving a SIGTERM. SIGTERMs are generated by container rescheduling, rolling deployments, or host maintenance. Services may request a grace period of up to five minutes between TERM and KILL. Any number of SIGKILLs is not normal, and developers should address them immediately. If a service isn't shutting down cleanly, then things are probably going wrong. It may mean that connections aren't being closed and are staying open until their negotiated session timeout. If nothing else, it means a service is not shutting down as quickly as expected.

Like OOM events, consider setting a warning threshold for more than one event in five minutes, and a critical threshold for more than three events in five minutes.

### 3. Excessive load-balancer connections

This event counts the number of connections made from a single proxy host (load balancer) to a service in a container. The default connection count limit is 5,000. After that, new requests to that service will return a 503 response to the client. Like OOM events, any number of 503 responses is cause for concern, so this alert should be set to fire well before it reaches the default count limit. The CF team encourages a warning threshold of 4,000 connections, and a critical threshold of 4,800 connections.

## Happy operators => happy customers

Our customers don't necessarily care how robust our health checks are, or whether we develop our services to self-correct for failures - they just want to use our platform the way our SLAs say they can. Nevertheless, we believe that architecting services for resiliency is a discipline, not a task, and that there is a direct correlation between operator restfulness and customer satisfaction. Hopefully these health check guidelines and monitoring and alerting examples help you think about ways to enhance your orchestration platform, so you too can build highly available services to delight your customers and keep your operators rested and healthy.
