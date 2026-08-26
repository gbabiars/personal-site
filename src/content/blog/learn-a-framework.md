---
title: "Sometimes You Should Learn Frameworks"
description: "There was recently a post that made the rounds on Twitter about not learning frameworks. While I actually agree with the general premise that trying to learn every framework is…"
pubDate: "2018-12-30T00:00:00"
---

There was recently a [post](https://sizovs.net/2018/12/17/stop-learning-frameworks/) that made the rounds on Twitter about not learning frameworks. While I actually agree with the general premise that trying to learn every framework is a bad idea, I also think that the idea of learning a framework for learning sake is valuable. As important as fundamentals are, frameworks exist for a reason and it is important to understand what problems they solve so you do not end up wasting your time or having to rewrite your app later.

### Learning new concepts

Many frameworks differ in the approach they take to solving similar problems. For example, JavaScript UI frameworks can be radically different even though they are solving how to build web UIs. Often times these concepts can be learned without actually building anything. Understanding the concepts behind a framework can help you know if that framework is worth exploring deeper.

Some examples of notable web framework concepts:

- React's virtual DOM
- Ember's routing
- Ember's convention over configuration
- Vue's single file components
- Svelte as a compile away framework

Understanding framework concepts can help you better understand the tradeoffs different frameworks make so that you can make more informed choices about which framework to use for a project.

### Learning new patterns

Beyond the conceptual value, many frameworks codify different patterns either through the framework itself or through community practices. These patterns can be valuable to learn because they can often be applied outside of that specific framework.

Some examples of notable web framework patterns:

- Unidirectional data flow
- Computed properties
- Higher order components
- Render props
- Code splitting

Learning patterns via a framework can be a great way to expand your knowledge base and make you a better developer in the framework(s) you already work in. For example, I first learned React while working with Ember and it actually improved the way I handled state in Ember.

### Appreciating developer experience

Different frameworks make different tradeoffs when it comes to developer experience. There has recently been an uptick in improved developer experience out of the box for many different UI frameworks (Ember CLI does not get enough recognition for being a catalyst here). This developer experience comes in two ways: ease of code maintenance and tooling to help you build apps.

For code maintenance, this can be:

- How easy is it to follow the data flow
- How easy is it to create and maintain functionality
- How flexible vs prescriptive is it about how apps are built

For tooling, this can be:

- Is there a CLI tool?
- Are there browser plugins?
- How easy is it to get a new app running
- How flexible is it for different needs

This is something that was overlooked for a long time, but is still evolving. Having good ergonomics for code and solid tooling can really affect the productivity of a team. For example on tooling, years ago when I was working with Ember, Ember Inspector and Ember CLI made building and debugging apps feel natural, but looking at AngularJS and React it was amazing the tooling gap that existed at the time. On the code side though, React made creating new components inexpensive (just functions) and opened new ways of composing that were difficult in other frameworks. Understanding the tradeoffs are important to be able to know what you are gaining and what you are giving up by choosing a particular framework.

### Learn only what is needed

With all that said, I don't think it is useful to invest large amounts of time learning frameworks. Frameworks come and go, but knowing what is out there and what are the tradeoffs is important. For me, this has meant waiting until a framework is clearly popular enough that it isn't going anywhere, reading through the basic concepts, and finally, if I care enough, build something simple in it to get a better feel for it. Most of the time you can get a feel for what the tradeoffs are without ever writing a line of code.

Don't bother spending time mastering an API which might change. Focus on learning the concepts, patterns and tradeoffs that come with frameworks. Finally, don't try to learn everything, but sometimes you should take the time to learn what makes them useful.
