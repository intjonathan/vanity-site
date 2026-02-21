---
title: "Dashboards for DevOps: Examples of What to Measure"
date: 2017-01-18
author: Jonathan Owens
---

*This article was originally published on the New Relic blog. See an [archival copy](https://web.archive.org/web/20230327234956/https://newrelic.com/blog/best-practices/dashboards-devops-measurement) at the wayback machine.*

So you've officially hopped on the DevOps bandwagon. You're working alongside your dev or ops counterpart, you're deploying more frequently, and it feels awesome to be able to work with more flexibility, collaboration, and agility. That is, until one of the execs at your company stops by and asks how the DevOps initiative is going and all you can say is "great!" without any data to back it up.

Without data, all you have is opinion. How much better to say, "Great! Our number of incidents went down, and we deployed twice as many times this month without any outages." You need to be able to prove your DevOps success—and that's exactly what a tool like New Relic can help you do.

Here are four examples of New Relic dashboards that you can use to show progress in your DevOps initiatives. With these in hand, you can be ready with a more substantial response to the "how's it going?" question the next time the boss comes around.

## Business performance dashboard

![Business performance dashboard](/assets/images/dashboards-for-devops/devops-1-1.jpg)

So let's start with a dashboard that begins with the customer. We'll call this our "business performance" dashboard and it should answer all the key questions an executive or business stakeholder will want answers to. "How many users do we have? What's our average revenue per user? Per hour? What kind of experience are we delivering to our customers?"

As you can see in the above example, using New Relic Insights we've created a dashboard that gives both technical and non-technical teams a simple, easy-to-understand look at the metrics they care about most. In this case, we're tracking page views, revenue, errors, response time, and [Apdex](https://docs.newrelic.com/docs/apm/new-relic-apm/apdex/apdex-measuring-user-satisfaction) among other things, which you can use as a starting point. But be sure to talk to your business stakeholders and the people who were involved in the decision to move to a DevOps methodology to understand what the company's goals are so that you can [create your own real-time dashboard](https://docs.newrelic.com/docs/insights/new-relic-insights/managing-dashboards-data/building-insights-dashboards) that shows the rest of the company the impact you're making on the business.

To build something like this, start with the Transaction and PageView events emitted by our language agents. These provide the bulk of the data shown here, like Errors, Response Time, and Apdex. To get Revenue, try attaching a [custom attribute](https://docs.newrelic.com/docs/agents/manage-apm-agents/agent-data/collect-custom-attributes) to your Transaction events during your checkout experience if you have one.

With this type of visibility, you can take credit for that bump in overall revenue ("Thanks to the fewer errors and improved response time starting here on this day and time, we've seen revenue numbers going up significantly"). And hopefully get a bump in your salary as a result!

## End-user and app performance dashboard

![End-user and app performance dashboard](/assets/images/dashboards-for-devops/devops2-1.jpg)

If you're using New Relic, you're likely already looking at the [New Relic APM Overview](https://docs.newrelic.com/docs/apm/applications-menu/monitoring/apm-overview-page) screen on a regular basis. But are you the only one?

Ideally, both your dev and ops teams would start the day with a level-setting look at how your application is performing. In this case, you'd want a dashboard that would collect information from New Relic's [APM](https://newrelic.com/application-monitoring), [Browser](https://newrelic.com/browser), and (if applicable) [Mobile](https://newrelic.com/mobile-monitoring) agents, so that everyone has visibility across the entire stack and understands the day's priorities to improve customer experience. As code changes are deployed to dev and test environments, these teams can see the impact on performance in real time.

In addition to code changes, you'll be able to quickly see APM-specific metrics like transaction times, Apdex score, error rate, and throughput, as well as Browser-specific metrics like page view load time, session traces, JS errors, and AJAX response time, or Mobile-specific metrics like crash rate, HTTP errors/network failures, HTTP response time, and more. Your ops teams should also be seeing these metrics, as their work on the servers can affect them. These should be familiar metrics to folks working lower in the stack, so rope them in too.

## Change events dashboard

![Change events dashboard](/assets/images/dashboards-for-devops/devops3-1.jpg)

Deployments are an obvious metric you'd want to track in a DevOps environment, but with New Relic (and our [Infrastructure product](https://newrelic.com/infrastructure), specifically) you can measure *all* change events that are happening in the system. For example, using the live-state event feed in New Relic Infrastructure, you could see when someone logged in and changed the packages—that's the level of detail you want when you're talking about a DevOps methodology. You want to be monitoring not just the application but the ground on which your application sits and the services that it uses.

So in the above example, we're monitoring AWS EC2 with the change events there in the right column. The beauty of using Infrastructure to monitor your [AWS environment](https://newrelic.com/partners/aws-monitoring) is that you can create dynamic dashboards and alerting using any key AWS attribute, whether that's role, tier, AZ, datacenter, or any custom EC2 tag. From your infrastructure and applications to your end-user experience, you can use New Relic to see every change in your stack so you can maintain optimal performance at scale.

Here's another example of how New Relic can help you stay on top of change events. In this case, we're looking at deployments:

![Deployments dashboard](/assets/images/dashboards-for-devops/devops4-1.jpg)

With New Relic's deployment analysis, history, and comparison, you can see the before-and-after picture of your app's performance when a change has been deployed.

## Alerts dashboard

![Alerts dashboard](/assets/images/dashboards-for-devops/devops5-1.jpg)

Another useful dashboard for DevOps teams is one that [tracks alerts](https://newrelic.com/alerts). In this example, we're feeding our PagerDuty alerts data into New Relic Insights to quickly identify trends and patterns. Monitoring on-call load in a programmatic and visual way can be very helpful for devs (who may not be used to getting paged for everything) because you can all look at this chart and realize, for example, that indeed the pages are five times worse than last week. (So they *did* have a reason to be whining...) It gives everyone a fast and easy way to pinpoint exactly where things got bad.

The real magic, though, happens when you start to correlate that alert with a specific change in the system. You leverage the data in New Relic APM to track deploys, top-five response times, throughput, database, or errors to find out exactly when and why the alerts started firing off more frequently.

At New Relic, we try to alert on the highest value metric (e.g., response time, as that would be a direct indication that customer experience is suffering) and then let the investigation flow out of the response to that alert, rather than responding to 50 alerts of all the things that went wrong at the same time that made the response time go bad. It's about finding that sweet spot, because while you could potentially set up an alert for all the hundred-plus moving parts in the system, rarely do you care about any of them in isolation. What you care about is when enough of them go wrong to make the customer start getting unhappy. So when it comes to picking an alert, start with customer impact: what's going to make a customer call in and complain?

Once you've picked those alerts, track their volume and severity just like any other metric. They're a very high-signal way to learn about where the problems in your systems lie, and whether they're getting better.

## Build your own dashboard

Now that you have a better idea of what to monitor, you're ready to create your own DevOps dashboards using New Relic Insights. If you don't have much experience with our analytics solution, be sure to check out the following resources to get started. You can learn how to build a dashboard, what type of data and widgets you can use, and more:

- [New Relic Documentation: Building Insights dashboards](https://docs.newrelic.com/docs/insights/new-relic-insights/managing-dashboards-data/building-insights-dashboards)
- [Blog: New Relic Insights Adds Even More Flexibility to Its Dashboarding Capabilities](https://blog.newrelic.com/product-news/insights-timepicker-everywhere-data-apps/)
