---
title: "Responsive Ember Components"
description: "Over the past several years, responsive design has become very popular in the web development community. It allows us to build for ranges of devices starting with mobile and up…"
pubDate: "2015-05-21T00:00:00"
---

Over the past several years, responsive design has become very popular in the web development community. It allows us to build for ranges of devices starting with mobile and up to large desktop screens. To do this, [media queries](https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Media_queries) have become a staple of building responsive stylesheets, allowing us to have conditional styles based on the size of the screen, typically checking for width. Media queries fall short when building large "apps" though. What we really need are [element queries](http://www.smashingmagazine.com/2013/06/25/media-queries-are-not-the-answer-element-query-polyfill/).

Before discussing how we can implement this in Ember, I'd like to take a moment to discuss why we need element query like functionality. I consider media queries great for designing things "in the large". By that I mean things like the app wide layout. What it doesn't do well is allow us to be responsive for things "in the small". By that I mean reusable components for our application. Often times the way a component should be styled is determined by the size of the containing element and not the size of the entire screen.

Let's look at a quick example that will highlight what I am talking about. Let's say we have a user profile component (not referring to anything Ember specific yet). When possible, we will have the user profile image be to the left and the text info, like name and email, to the right. Let's say the minimum amount of width we want for this layout is 400px. When we are under 400px, we want to have the profile image centered and have the profile information to wrap under it.

![](/content/images/2015/May/profiles.png)

In many cases this is simple enough to do with media queries by setting a breakpoint, but let's say our layout is a little more complex. For example, we may have a paned view where this profile might be used. In this case, the screen might be larger, but the container is not. We could do it with media queries, but we'll end up having odd ranges that are brittle if other things change. In this case we really need to know the size of the element itself in order to style it.

### Simulating Element Queries in Ember

To solve the need for element queries in our Ember components, we need to track the width of the element. We'll then use this width for a computed value which can be used in a conditional class. In the end we'll have default styles which are the collapsed version then dynamically expand when the width of the component's element is larger than 400 pixels.

Our component JS code:

```js
// user-profile.js
export default Em.Component.extend({
  classNames: ["user-profile"],
  classNameBindings: ['isExpanded:user-profile--expanded'],

  didInsertElement: function() {
    this._resizeHandler = function() {
      this.set('width', this.$().innerWidth());
    }.bind(this);
    $(window).on('resize', this._resizeHandler);
    this._resizeHandler();
  },

  willDestroyElement: function() {
    $(window).off('resize', this._resizeHandler);
  },

  isExpanded: Em.computed("width", function() {
    return this.get("width") > 400;
  })
});
```

Our component template:

```handlebars
<img src="{{imageSrc}}" class="user-profile__image" />
<div class="user-profile__info">
  <h3>{{user.name}}</h3>
  <h5>{{user.email}}</h5>
</div>
```

Finally our styles:

```css
.user-profile {
  text-align: center;
}

.user-profile--expanded {
  display: flex;
  align-items: center;
  text-align: left;
}

.user-profile__image {
  width: 150px;
  height: 150px;
}

.user-profile__info {
  padding: 0 15px;
}
```

### Refactoring to a Mixin

While that component worked, we would really want to do that for every component. So let's move that common logic out. We'll actually have two steps. First, we'll move our resize handler out to a common object that can be used as a singleton across our app.

```js
// browser.js
export default Em.Object.extend(Em.Evented, {
  init: function() {
    this._super();
    this._handleResize = function(e) {
      this.trigger('resize', e);
    }.bind(this);
    $(window).on("resize", this._handleResize);
  }
});

// initializer
application.register('browser:main', Browser, { singleton: true });
application.inject('component', 'browser', 'browser:main');
```

So here we're injecting this onto all components as `this.browser`. Now let's move the width tracking logic into a mixin.

```js
// responsive-component.js
export default Em.Mixin.create({
  setupResizeHandler: function() {
    this._resizeHandler = function() {
      this.set('width', this.$().innerWidth());
    }.bind(this);
    this.browser.on('resize', this._resizeHandler);
    this._resizeHandler();
  }.on('didInsertElement'),

  teardownResizeHandler: function() {
    this.browser.off('resize', this._resizeHandler);
  }.on('willDestroyElement')
});
```

Finally, we'll clean up our component.

```js
// user-profile.js
export default Em.Component.extend(ResponsiveComponentMixin, {
  classNames: ["user-profile"],
  classNameBindings: ['isExpanded:user-profile--expanded'],

  isExpanded: Em.computed('width', function() {
    return this.get("width") > 400;
  })
});
```

Now we can easily implement this across our app without all the extra boilerplate.

### Warning

This is only a pattern I have been playing with recently and there are some caveats to it. First, there are likely optimizations and things I have left out like wrapping code in run loops to perform immediately when browser events happen. Secondly, this only updates on the window resize, so if you have things like expanding panels, you would need to hook into those events to update the width. Finally, there may be performance implications of doing this, particularly with large lists, that I have not explored yet.

This pattern I feel is just a starting point in order to build out components where styles are self contained. There are a lot of possibilities on where we could take this in order to make maintaining component driven apps easier.
