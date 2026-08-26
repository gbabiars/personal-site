---
title: "Can We Stop Saying Single Page Apps?"
description: 'If there is just one phrase in the web development community that I wish could just go away, it would be "Single Page Apps". My big problem with the term can be misleading and…'
pubDate: "2013-09-15T00:00:00"
---

If there is just one phrase in the web development community that I wish could just go away, it would be "Single Page Apps". My big problem with the term can be misleading and uninformative. It gives no insights into the technology behind the application. Like HTML5, the term "single page app" is more of a marketing term than a technical term.

### What's in a name?

So what exactly is a single page app or SPA? Typically this means that there is one server rendered page, with the rest of the content being dynamically generated with ajax requests and javascript. With this description, single page app is pretty acurate. So then what's the problem?

### And then there were two…

Let's take a pretty common scenario of having an application that requires logging in. Let's say that our login page is actually a separate page, but the rest of the app fits the single page model. Does this mean we no longer qualify as a single page app? What do we call our app then?

Even if we only have one server rendered page, most SPAs have client side navigation using either push state or hashes. Really, we have multiple pages, but the routing is done on the client rather than the server. In this instance, describing something as single page is very misleading.

### It shouldn't be this hard

There are a lot of alternative names out there. Ember uses the term "ambitious web applications" which is pretty good. There's also "javascript web applications" or "client side javascript application" and many more. These help a little bit, but can't we do better?

When we are building traditional server side MVC applications, we typically describe them using the framework they are built on top of. When someone talks about their rails app or Django app, we have a solid understanding of what they are talking about. Why don't we just do this on the client side? When someone talks about an Ember app or an Angular app we'll have a general understanding of how it was built.

In the end we're just building web applications, so let's just leave it at that.
