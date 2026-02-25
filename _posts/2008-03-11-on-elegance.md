---
title: "On Elegance"
date: 2008-03-11
author: Jonathan Owens
---

Lately I've been working on a new media serving backend for Treemo. We've outgrown the features on our existing setup, and since the vast majority of our hits are content downloads, it's time to optimize those.

One thing that we've discovered needs changing is the URL structure for our content. Most large content-focused sites like Flickr, Fotolog, or Imageshack use an unguessable portion of the URL to enforce visibility restrictions. This allows easy connection to a CDN, as you can simply pass out those URLs to a hosting service secure in the knowledge that they will only be presented to the appropriate people. Livejournal is an exception, they do authorization and byteserving on their balancers. We copied this setup initially, but as is the case with many Livejournal tools, we found we had to really go whole hog copying the rest of their setup (in this case, perlbal) to make it useful. So we're changing our approach.

Obviously changing your URL scheme for embedded content is a rather major migration, and not to be done lightly. Among the many issues that have arisen during this process is what the URLs should look like. I spent a lot of time evaluating the Flickr URL scheme, as they're the biggest and we were looking to model their architecture instead of Livejournal's this time. One thing I came to appreciate about their scheme is the elegance. Here is the basics:

```plaintext
#
# Typical usage
#

https://live.staticflickr.com/{server-id}/{id}_{secret}_{size-suffix}.jpg

#
# Unique URL format for 500px size
#

https://live.staticflickr.com/{server-id}/{id}_{secret}.jpg

#
# Originals might have a different file format extension
#

https://live.staticflickr.com/{server-id}/{id}_{o-secret}_o.{o-format}

#
# Example
#   server-id: 7372
#   photo-id: 12502775644
#   secret: acfd415fa7
#   size: w
#

https://live.staticflickr.com/7372/12502775644_acfd415fa7_w.jpg
```

_See [Flickr API docs](http://www.flickr.com/services/api/misc.urls.html)_

The longer I worked on various combinations of schemes that would give us the structure we wanted, the more I realized that their scheme included everything you needed and nothing you didn't, and did so in under 80 characters. Making something that nice or nicer became a goal.

I got to the point of evaluating how to alter our object model to accommodate the new information that the filesystem would need to back up the new scheme, and darn if it wasn't hard. Adding the data I wanted was going to be expensive. There were easier schemes I could use involving long hashes or other things that would be just as secure, just as functional. But they'd be huge. And something in me didn't like that, and a few halfhearted dust-ups with my officemates found me unable to articulate why.

Part of it felt like well, Flickr has already proved this scheme works. They've learned lessons about it that we don't haven't, so let's not reinvent the wheel here. But that doesn't get me very far. Other, easier schemes would work, and who cares what the URLs look like?

And I couldn't escape the fact that I do, and I had to leave it at that.

But the more I thought about it, stronger I felt that not only should a new scheme be functional, but it should be *nice*. It should bear the mark of a system made with care for *people*, not tools. I think the ability to make this small part of the system exceed requirements in an elegant and surprising way became important, because if all I do is make tools that work, I get bored. Anyone can write code that does what the customer wants. The magic happens when we make code the customer doesn't realize he wants. You might not realize that nice URLs make the system more humane to use, but when they're obnoxious, it's noticed. "Huh," your users will say, "that's funny-looking." Not a thought that should be going through your users' minds, even for the half-second their subconscious will think about it.

I think we as developers have a lot of responsibility to think like a user even - maybe even especially - when it makes our jobs harder. Thinking for them helps us develop products that exceed their expectations, which allows teams to gel, which is the only way great products are made.
