---
title: "The Myths of Backbone.js"
description: "When it comes to building javascript centric web apps, it's hard to argue with the popularity of Backbone. It has played a big part in shifting client side javascript apps from…"
pubDate: "2013-08-01T00:00:00"
---

When it comes to building javascript centric web apps, it's hard to argue with the popularity of Backbone. It has played a big part in shifting client side javascript apps from being jQuery spaghetti to well structured, more maintainable pieces of software. It has helped move developers away from "truth in DOM" and towards keeping record stores in the javascript application itself. While it has done so much, an amazing achievement for such a relatively small library, there is a lot of misinformation about Backbone that exists in the web community.

As a preface, I do use Backbone on a daily basis and both appreciate it for the simplicity and functionality it provides, while also wishing for more at times. Developers who have used Backbone on a regular basis and have built non-trivial applications have also felt this pain. These are simply some of the myths I feel exist about Backbone.

### It is a framework

If you read any blog posts or see any getting started guides, you'll more than likely hear Backbone referred to as a "framework". This is something I disagree with strongly, as Backbone is much more of a library than a framework. This is not a knock on Backbone, in fact Backbone is a library that gives you the pieces to create your own framework. It has many of the same components you'd expect to see in a framework (models, views, router), but if you take away the router, you'll see what you are given are building blocks. Frameworks will wire things up for you, whether through configuration or convention, but Backbone leaves it up to you to figure out how to wire up your application. This allows it to be very flexible, but also places great responsibilities on the developers using it.

### It is easy to learn

This is one of the most common claims when people evaluate Backbone. There is no question that the core of Backbone, particularly the models, collections and views, is easy to learn. The difficulty instead lies in figuring out how to get all the pieces to work together. Backbone's difficulty ramps up as your application get's more complex. How do you manage child views? What about aggregates that depend on multiple collections? What about cleaning up your views to prevent zombie events? Backbone's unopinionated approach gives you very little guidance on how to solve these situations, therefore it is up to the developer to learn patterns used in the Backbone community or try to figure it out themselves. One great thing about Backbone's popularity is that most of these problems are solved and many libraries exist that do the work for you (i.e. [Marrionette.js](http://marionettejs.com/)).

### It is lightweight

The library itself is only 6.3k, but it lacks any boilerplate code. This means you will either be writing this yourself or using 3rd party libraries. Much of this depends on the use case, but you can often end up with the same total size application with Backbone as you would had you used one of the larger frameworks.

### It is better/worse than…

Insert a framework or library here. Backbone vs. Angular? or vs. Ember? What about compared to Knockout? It has become very common to see comparisons between the different libraries and frameworks looking for a winner. Unfortunately most of these comparisons are apples and oranges. Besides that, there is plenty of room for many solutions to the wide range application needs that exist. It is important to understand the different features and pitfalls of using each solution so that you or your team can make the best choice for your projects.

### …but in the end…

Backbone is still a good choice when building client side javascript applications, but you should understand what it gives you and what you have to do yourself. Luckily Backbone's popularity has led to a large plugin ecosystem that solve most of the common problems. In the end, if you are going to use Backbone, expect to have to make some serious architectural decisions and write boilerplate code.
