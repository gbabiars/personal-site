---
title: "Using Ember Components as Content Wrappers"
description: "If you're using Ember, hopefully you've come to appreciate how powerful components can be when building an application. Many times, components are simply black boxes that take…"
pubDate: "2014-12-22T00:00:00"
---

If you're using Ember, hopefully you've come to appreciate how powerful components can be when building an application. Many times, components are simply black boxes that take in values/bindings and emit actions to handlers. They allow us to build reusable pieces of functionality which serve as the building blocks for our application.

From my perspective, there are two types of components: black boxes and wrappers. By black box, I mean the components that completely encapsulate all functionality and template within the component. The only way to access these are by passing properties to the component and describing how the actions it emits are mapped to the outside context. Examples of this would be a date picker or user profile display.

```js
{{date-picker value=date onChange="dateChanged"}}
```

```js
{{user-profile user=user}}
```

While these are extremely powerful and in my experience the most common, Ember allows for a second type of component which are wrappers around content. These serve as a way to have reusable pieces of functionality while the inner template is unique to the specific use. This is done using `{{yield}}` in the component's layout (or the default if you have no layout template defined).

To highlight the benefits of this, let's look at a couple examples of how they can be used. First, let's look at a common situation in most applications: forms.

#### Example: Form Component

Most forms have common charactersitics: they allow editing inputs, submit the changes, validate the inputs and either do some action or display validation errors. Let's assume we are doing this on a controller. It might look like this:

```js
// in our controller
validationErrors: [],

actions: {
  submit: function(model) {
    return model.validate()
      .then(function() {
      	return model.save();
      })
      .catch(function() {
        // generate messages on this.validationErrors
      });
  }
}

// in our template
<form {{action "submit" model on="submit"}}>
  {{#each error in validationError}}
    <div>{{error}}</div>
  {{/each}}

  // rest of form
</form>
```

A couple of explanations on what is going on here. This assumes we are using Dockyard's [Ember-Validations library](https://github.com/dockyard/ember-validations) which tracks error state directly on the model. I've omitted extra logic to do this because it is not pertinent to the concept. While this works well, there are a couple of issues. First, since controllers are singletons our `validationErrors` must be managed in places like the route's `setupController` hook and on save so we don't accidently have old messages displayed. Secondly, this is a nightmare for reuse. To reuse this on multiple controllers, we are going to have to duplicate code in both controllers and templates.

Rather than duplicate, we can create a form component which will wrap our form content, take care of the validation, display error messages, etc. The best part is we can still have our custom template inside and call out to our custom logic upon validation.

```js
// component definition of form-for
Em.Component.extend({
  tagName: 'form',
  validationErrors: [],

  submit: function() { // event, not action
    var model = Em.get(this, model);
    return model.validate()
    	.then(function() {
      	this.sendAction('onSubmit', model);
      }.bind(this))
      .catch(function() {
        // generate error messages on this.validationErrors
      });
  }
});

// form-for layout
{{#each error in validationErrors}}
  <div>{{error}}</div>
{{/each}}
{{yield}}

// calling template
{{#form-for model=model onSubmit="submit"}}
  // form markup, including buttons, excluding error summary
{{/form-for}}

// in our controller/outer context
actions: {
  submit: function(model) {
    return this.save();
  }
}
```

Let's look over what has changed. Our form-for component takes in the model and an action that is called for onSubmit. The layout template includes the error summary display at the top and then yields below for the spefic form template. When the form is submitted, the component will check if valid and if so, will call trigger the onSubmit action, which will call the controller's specific code. If the model is not valid, the error messages will be generated. We've omitted some logic for resetting errors after submit, but we don't need to worry about state being persisted when we navigate away and back because the component is not a singleton. We always get the default empty array when we create the component.

We can now use this form component across our application for any forms we need. This cleans up our controller/outer contexts which call these forms to focus on just what success logic they need to call by ignoring invalid submissions.

This example component is pretty naive. For more info on form components check out [Awesome Ember.js Form Components](http://alexspeller.com/simple-forms-with-ember/) or [Ember EasyForm](https://github.com/dockyard/ember-easyForm).

#### Example: Auto Resizing Section

A specific use case we had in our application came about when we needed to have a section which would at least fill the page. The page had a header and footer, and the content section should fill the remainder of the height if the content was not already greater than that height. This could not be done at the route level because there was no common shared parent where all child routes shared this behavoir. There are also multiple things that can cause a section to resize: route changes and window resizes.

The solution ended up being very simple: a component which contained all the resize logic and would wrap our content.

```js
// auto-fill component
Em.Component.extend({
  $window: $(window),

  didInsertElement: function() {
    var component = this;
    this.router = this.container.lookup("router:main");
    fillHeightIfNecessary(component);
    this.transitionHandler = function() {
        Em.run.scheduleOnce("afterRender", function() {
            fillHeightIfNecessary(component);
        });
    };
    this.router.on("didTransition", this.transitionHandler);
    this.resizeHandler = function() {
        fillHeightIfNecessary(component);
    };
    this.$window.on("resize", this.resizeHandler);
  },

  willDestroyElement: function() {
    this.router.off("didTransition", this.transitionHandler);
    this.$window.off("resize", this.resizeHandler);
  }
});

// in our template
{{#auto-fill}}
  specific markup goes here
{{/auto-fill}}
```

The component does very little. When it is first inserted, it resizes based to the required height and sets up listeners on the two events we care about. I've omitted the calculation function, but it simply compares the container height to the min height and sets CSS values if needed. The best part about this code is that it is very specific but can be used in multiple places across our application.

#### Benefits

Hopefully these examples are enough to convince you how powerful components can be as general wrappers around specific content. They give us a great amount of reuse without sacrificing customization or creating too heavy of abstractions.

Compared to controller backed templates, they are much easier to reuse across the application. On top of that, the lifecycle hooks (willInsertElement, didInsertElement, willDestroyElement) give us a much cleaner way to manage state than in route hooks to update a controller. Furthermore, because components are created as they are needed, we can rely on the defaults being correct when we navigate away and come back to a page, unlike controllers by default.

#### Limitations

While Ember Components are pretty good, they currently have some limitations. First, we can only have one `{{yield}}` per component. This makes it a little more difficult to have multi sectioned components like a panel component that has a header, body and footer where each section can have customizable markup. To get around this, you can build nested components which rely on `parentView` to access the parent component.

```
{{#panel-for model=model}}
  {{#panel-header}}{{/panel-header}}
  {{#panel-body}}{{/panel-body}}
  {{#panel-footer}}{{/panel-footer}}
{{/panel-for
```

Check out Ryan Florence's [talk](https://www.youtube.com/watch?v=nVTXJyyBxQc) at last year's EmberConf for more on this technique.

The second limitation is the inability to access properties on the component from inside the yielded section of our template. This can be beneficial when we want to customize that section based on some calculated state of the wrapper. There are ways to work around this, mainly binding the value to something on the parent context, but it feels hacky. I believe this will be solved with the introduction of **Block Params**. Check out the [RFC](https://github.com/emberjs/rfcs/blob/master/active/0002-block-params.md) for more info on this.
