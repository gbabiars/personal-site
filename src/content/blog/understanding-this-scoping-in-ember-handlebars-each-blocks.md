---
title: "Understanding {{this}} scoping in Ember Handlebars {{each}} blocks"
description: "One of the more confusing parts of using Ember is understanding how {{this}} is scoped within an {{each}} block in your templates. The reason this can be so confusing is…"
pubDate: "2013-11-28T00:00:00"
---

One of the more confusing parts of using Ember is understanding how `{{this}}` is scoped within an `{{each}}` block in your templates. The reason this can be so confusing is because the scope of the {{each}} block can change depending on what you are iterating over and how your controller is set up. I will cover the three main use cases and discuss what `{{this}}` resolves to in each.

###Case 1: {{#each model}}
This is the simplest case in which we are iterating over an array without explicity declaring what our variable name is. The template will look like:

    <ul>
    {{#each model}}
        <li>{{this.name}} - {{controller.message}}</li>
    {{/each}}
    </ul>

Our controller will look like:

    App.PersonController = Em.ArrayController.extend({
        model: [
            { name: 'John Doe' },
            { name: 'Bob Jones' }
        ],
        message: 'Hello'
    });

In this case, `{{this}}` will be set to the current person in the list that is being iterated over. The big thing to note here is that our context has changed from outside of our each block, where the context typically is the controller. We can still access the controller inside the each block by using `{{controller}}`. Note that we put `{{#each model}}`, but we could have used `{{#each controller}}` as well since we have an ArrayController which proxies the model property. In a later case we'll see where this is an issue.

###Case 2: {{#each person in model}}
In this instance we are explicity declaring the name of the current item. Our template will look like:

    <ul>
    {{#each person in model}}
        <li>{{person.name}} - {{message}}</li>
    {{/each}}
    </ul>

Our controller is exactly the same as Case 1. The difference here is that {% raw %}{{this}}{% endraw %} in our {% raw %}{{each}}{% endraw %} block is going to be the same as our context outside of our each block, which is typically our controller. So we can display properties on the controller in our loop without having to explicitly say they are coming from the controller. This is my prefered method of displaying a list of items because it avoids a context switch which makes it more difficult to read.

###Case 3: {{#each controller}} with itemController
In our last case, our template will be very similar to Case 1, with the exception that we will use an itemController. The template will look like:

    <ul>
    {{#each controller}}
        <li>{{this.name}} - {{parentController.message}} - {{selected}}</li>
    {{/each}}
    </ul>

Our controller and itemController will look like:

    App.PersonController = Em.ArrayController.extend({
        itemController: 'personItem'
        model: [
            { name: 'John Doe' },
            { name: 'Bob Jones' }
        ],
        message: 'Hello'
    });

    App.PersonItemController = Em.ObjectController.extend({
        selected: function() {
            return this.get('name') === 'John Doe';
        });
    });

The big difference with this is that we are able to add controller logic to each item in the list. This is often important because we don't want to put transient data in our model, but we doesn't make sense in the list controller. Another important thing to note here is that `{{controller}}` is set to our itemController instead of the controller outside of our each block. To access this, we can use `{{parentController}}` which is automatically set on an itemController.

###Summary
As you can see, each blocks in Ember templates can be confusing because of the variable context. Understanding how this scoping works goes a long way to being able to select the one that fits your use case.
