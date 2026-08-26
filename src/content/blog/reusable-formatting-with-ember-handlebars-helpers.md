---
title: "Reusable Formatting with Ember Handlebars Helpers"
description: "One of the great things about Ember is that it uses Handlebars as its default template engine. Handlebars allows you to create a very declarative yet powerful UI through it's…"
pubDate: "2013-06-16T00:00:00"
---

One of the great things about Ember is that it uses Handlebars as its default template engine. Handlebars allows you to create a very declarative yet powerful UI through it's built in helpers like `{{each}}` and `{{if}}`. Ember has also extended Handlebars with helpers like `{{view}}` and `{{render}}` as well. While these helpers go a long way, Ember allows us to easily create our own custom helpers. This allows us to build a UI that is very expressive while also DRYing up our application. A great way to utilize this feature is by moving common formatting of values and types into helpers.

### Date Formatting

To demonstrate how this can be beneficial, let's take a look at a pretty common task in applications: formatting dates. The formatting logic can often be spread throughout the application without consistency. We can clean this up by moving the formatting out of the models, controllers and views and into our custom helpers which we can reuse in different contexts.

For this example, we'll take the idea of a simple bulletin board system with Posts and Users. The application will have three views: list of Posts, Post details and User details. Our models will look like this:

    App.User = Em.Object.extend({
        id: null,
        name: '',
        createdAt: null,
        lastOnlineAt: null
    });

    App.Post = Em.Object.extend({
        id: null,
        title: '',
        body: '',
        createdAt: null,
        lastUpdatedAt: null,
        lastReadAt: null,
        authorName: '',
        authorId: null
    });

For each post and user we have two dates. We want to render these dates in our templates.

### Let's Update our Models

Before we jump into helpers, let's handle the formatting within our models. We can create computed properties on the model, such as:

    createdAtPrettyFormat: function() {
        return moment(this.get('createdAt')).format('MMM Do YYYY h:mm A');
    }.property('createdAt')

but we'd quickly be duplicating our code. First, if we decide we want to have the same date formatted differently in different views (i.e. having it without the time on the posts list, but with on the post details) we would have to create a different computed property for each format. Secondly, we may have the same format for multiple dates and we would have to create different computed properties for each.

### Helpers to the Rescue

A better solution is to move the formatting to a Handlebars helper. For this example we are going to create two different helpers, one for relative dates and one for the full formatted date. First we need to register our helpers:

    Em.Handlebars.helper('relativeDate', function(value) {
        return moment(value).fromNow();
    });

    Em.Handlebars.helper('prettyDate', function(value) {
        return moment(value).format('MMM Do, YYYY h:mm A');
    });

The helper method takes in two parameters: the name of the helper and and the value that we want to format. Typically the logic here should be very simple, in this case we are just returning the date value as a formatted string.

Next, we can use utilize these helpers within our templates.

Posts List:

    <ul>
    {{#each model}}
        <li>{{#linkTo post this}}{{title}}{{/linkTo}} - {{relativeDate createdAt}}</li>
    {{/each}}
    </ul>

Post Details:

    <h3>{{title}}</h3>
    <p>Posted by: {{#linkTo user authorId}}{{authorName}}{{/linkTo}} at {{prettyDate createdAt}}</p>
    <p>{{body}}</p>
    <p>Last Updated {{relativeDate lastUpdatedAt}}</p>

User Details:

    <h3>{{name}}</h3>
    <p>User since {{prettyDate createdAt}}</p>
    <p>Last online {{relativeDate lastOnlineAt}}</p>

Even though these are very simple views, we have gained a lot of reuse and provided consistency to the formats. An added benefit is that the markup is very readable and obvious what format each date will be rendered as. And the best part is that the values are bound, so when a date is updated in our model the formatted date is updated too.

### Number Formatting

Formatting of numerical values is another place where helpers make sense, whether it is currency, percentages or placing commas. Here an example of a helper that displays as percent:

    Em.Handlebars.helper('percent', function(value) {
        return value + ' %';
    });

### Default Values

Another scenario is when we want to display a default message or value in place of a null or empty value. This is easily done using a helper like the following:

    Em.Handlebars.helper('message', function(value) {
        if(value) {
            return value;
        } else {
            return 'There is no value to display.';
        }
    });

### And More...

There may be many more domain specific use cases in your applications as well. The best part is that they are usually trivial to implement.

The repo for the date examples can be seen [here](https://github.com/gbabiars/bulletin-board-ember-helpers).

For more information on Handlebars helpers, check out the [documentation](http://emberjs.com/guides/templates/writing-helpers/).
