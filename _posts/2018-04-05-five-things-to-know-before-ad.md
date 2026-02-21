---
title: "Five Things to Know Before Adopting Microservice and Container Architectures"
date: 2018-04-05
author: Jonathan Owens
---

_Originally published on [The New Stack](https://thenewstack.io/5-things-to-know-before-adopting-microservice-and-container-architectures/). See an [archival copy](https://web.archive.org/web/20180718062311/https://thenewstack.io/5-things-to-know-before-adopting-microservice-and-container-architectures/) on the wayback machine._

As lead site reliability engineer (SRE) for New Relic's Container Fabric project (our internal container orchestration and runtime platform), I spend a lot of time with existing and potential customers answering questions about how we use and manage containers to create a platform composed of dozens of microservices.

We definitely consider ourselves early adopters of containers, and we started packaging services in them almost as soon as Docker released its first production-ready version in the summer of 2014. Many of the customers I talk with are just now beginning — or thinking about beginning — such journeys, and they want to know everything we know. They want to know how we make it work, and how we architected it. But part of the process, I like to stress, is that they need to know what we learned from where we struggled along the way.

With that in mind, here are five key takeaways I'd like to share with anyone pondering containers and microservices:

## 1. Never Stop Developing

Take your adoption project seriously, and treat it like a product. Give it a name, some internal branding even and a clear product vision. It should be managed and given a life.

Our current version of Container Fabric isn't our first attempt at homegrown orchestration and delivery. We built our original version in 2014, and once we got it as far along as we thought it needed to go, we just stopped developing it. We didn't have a full vision for a deployment platform as a product that our developers would enjoy using, and built something that satisfied only our need for a consistent abstraction layer.

So two years went by before we began to reinvest in our container platform. Our first version didn't stop being useful, but we failed to capture the many incremental improvements that emerged during Docker's rapid development phase.

One of our longest-running customers went through a similar container orchestration/microservices journey over the same time period, and they are much farther along than we are even today. When we asked them how they'd gotten so far, their answer was painfully simple: "We just never stopped working on it."

## 2. Build It Piece by Piece and Start with the Early Adopters

When you're shifting to containers, adopting a new technology (like Kubernetes) doesn't mean you jump right into the deep end and move your entire production fleet into giant highly available clusters. Isn't it easier to build things piece by piece?

When we started with Container Fabric, we constrained our roll-out to the simplest use case we could serve: stateless HTTP services — no persistence, no non-HTTP protocols and only one deployment scheme. This reduced the platform's feature footprint to one we knew we could serve effectively in a reasonable timeframe.

This was important because a container scheduling platform doesn't offer everything. We still needed to monitor the platform, deploy changes to it, handle secrets, configure automatic load balancing and capture logs — all the things that come with a rich service platform. Constraining the target service type made it easier for us to reason about the platform's components.

So if you're certain container orchestration is what you need, look hard at what the container platforms offer, and think about what's missing. What will you have to build on top of that platform to support your particular services and your infrastructure context? This will help you choose which component to focus on first.

Also, understand the culture and availability of your teams. Who are the early adopters in your company? What teams are primed for shifting into a new paradigm? What teams are building services that would fit well into a microservices architecture? What teams are stuck with legacy monoliths and need more time, planning and experimentation?

## 3. One Size Does Not Fit All

When it comes to containers and microservices, some systems just don't fit the model, and it's easier if you recognize that from the start. The technology in new systems is often so new that the effort to port older systems can be more trouble than it's worth.

Our platform continues to serve primarily HTTP services, but we're also starting to handle more stream processing services. We still don't do persistent storage or databases. Some parts of the New Relic platform may take years to move onto Container Fabric if we ever move them at all. And that's OK! Container platforms don't have a monopoly on good deployment practices.

The latest technologies and architectures are all about fast iteration and decentralized control, so if those things aren't crucial to your business goals, just let those old legacy systems chug along into retirement until you have a logical business need to migrate them.

## 4. Technology Changes are Organizational Changes

It's easy to start using a technology before thinking about how to use that technology. New architectures typically require new architectural knowledge and new ideas about how to deploy systems. The microservices pattern affects the architectures of both software and the organizations that use it.

As New Relic teams moved their services onto Container Fabric, their operations workload typically dropped. In some cases teams shed a significant amount of toil, freeing them to work on other projects they found more rewarding. This is the best case of automation removing the burden of toil, and we're thrilled to see it. Teams that have migrated get more freedom to build software which delights our customers.

Organizations considering a move to a container and microservices architecture should ask themselves the following questions:

- What product does your operations group provide to developers and what abstraction layer does that product use?
- Is that the right one for your business or are containers a better fit?
- Are containers so much better that you're willing to change the abstraction, and therefore the entire product your operations team offers, in order to use them?
- Are you ready to create new roles to manage the scale and reliability of this new abstraction?

No organization changes like this overnight. The journey from an idealized new architecture to the first production deploy requires changing many minds and creating new processes — which isn't always fun.

## 5. Remember, Building Software is a Human Experience

A rich, complex container scheduling platform takes hundreds of person-years worth of experience to build, maintain and scale. These platforms and architectures require you to accept new contexts and require that your organization accept trade-offs in its design as you adapt it to fit within your use case. Adopting a containerized microservices architecture is a big job that will likely change the balance of work across all your teams; it will affect the way they spend their time; it may sometimes affect the happiness they experience in their jobs. Unless you truly consider these human aspects of platform adoption, the migration will be far more difficult than it needs to be.

Because of our organizational structure, we've been well-positioned for a successful migration. Each service in New Relic is owned by one team — they plan, program, deploy and operate it. This structure has been in place for years, and the Container Fabric platform makes the work of these full-owner teams even simpler, letting them move up the layers of operations abstraction and closer to business goals.

The promise of containers and especially of their scheduler platforms often reads as "this is how everyone should develop software — it's full of magic." And that's often true — container platforms do democratize some radically powerful capabilities, and they are worth taking seriously. But remember that they all solve a platform productization problem in an opinionated way. You need to rationalize if those opinions can also be your own.
