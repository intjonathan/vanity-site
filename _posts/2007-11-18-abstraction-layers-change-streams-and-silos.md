---
title: "Abstraction Layers, Change Streams, and Silos"
date: 2007-11-18
author: Jonathan Owens
---

Being a developer, I tend to see people organization through the same lens as systems organization.

In a software system, every interface forms an abstraction layer. Every developer working within that system designs APIs, as this [helpful video](http://video.google.com/videoplay?docid=-3733345136856180693) points out. Good APIs have the qualities outlined in that video, that of being:

1. Easy to learn
2. Easy to use, even without documentation
3. Hard to misuse
4. Easy to read and maintain code that uses it
5. Sufficiently powerful to satisfy requirements
6. Easy to evolve
7. Appropriate to its audience

The first five items on the list are best accomplished by ensuring that the API presents a consistent level of abstraction. All its functionality should address the same set of concepts. We'll pick on PHP's Iterator interface:

```php
next()
prev()
valid()
rewind()
```

All these functions deal precisely with items in an iterable object. An example of an API that didn't do this might look like the following:

```php
$user->getHeldItems()
$user->holdItem()
$user->syncProfileData()
$user->checkoutItems()
```

If you've spent more than 10 minutes of your life writing code, that third method should fall with a loud thunk .

Making an API easy to evolve starts with change stream encapsulation. A change in the details of implementation can generally be ignored by higher-level consumers of the API. So over time, as that method evolves, its change stream will be hidden by its interface.

Organizations must also create the same abstractions, because nobody has the time to keep track of every single thing in the organization. Our office manager needs to not care how the DHCP server is configured, she just needs to know she can get to the Ikea site and get us some desks. Anything else is a distraction. For a developer, this is writ quite large, as the tools I require are typically much more complicated than a web browser. Maintaining groupware like Exchange, for example - not trivial. Babysitting a build process is another common one. I need that stuff to work so I can write good code, but I must spend almost none of my time doing it. So teams get created, presenting an organizational communication interface ("OPI"?).

And that's all well and good, but what happens when that boundary lies across a layer of the software? For example, when I rely on a RewriteRule set in Apache by our sysadmins? I have to communicate across a layer of both organizational and application-layer abstraction. And that presents a problem, because any organization-layer abstraction necessarily restricts the clarity of communication. I don't necessarily work with our sysadmin on Apache most of the time, so I don't have a clear idea of how it intertwines. That's a problem when I'm trying to design something that might interact with that subsystem.

So to fight this, organizational abstraction layers need to have the same careful consideration given them as a good API, and meet all 7 of the qualities listed at the top. Everything from process methodologies to groupware have to be evaluated in this way, because to solve the communication problem that is software, the ideas crossing those boundaries should be as fluent as possible.
