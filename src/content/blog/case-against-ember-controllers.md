---
title: "The Case Against Ember's Controllers"
description: "One of the most prominent parts an Ember application are controllers. Controllers allow you to add logic and hold data that back your templates. For small applications, this is…"
pubDate: "2014-09-06T00:00:00"
---

One of the most prominent parts an Ember application are controllers. Controllers allow you to add logic and hold data that back your templates. For small applications, this is simple enough and works well. But as your application begins to scale, the use of controllers becomes a burden on the maintainability of your application.

In this post, I will break down the major issues that hurt the scalability of controllers. I will also discuss other Ember mechanisms that overcome these issues.

### Issue 1: Runaway Growth

Let's start by thinking about an actual page in the application I work on. This is a page that displays the detail information of a contact in our application. It started simple enough by just having the model. Then some computed properties were added for display logic. Next it was made so we could toggle between editing and detail view, which meant extra state information and action handlers. New requirements had us add tabs with related information like groups and activities, which also had extra logic and data. With all the extra functionality, our controller ballooned in size and was no longer focused in it's functionality. It became the place to house all the logic and data on the page, so as the page grew our controller grew.

    App.ContactDetailController = Em.ObjectController.extend({
      // 30 or so lines of code with properties and logic for handling groups

      // 30 or so lines of code with properties and logic for handling activities

      // 20 or so lines to handle contact specific logic

      // 15 or so lines to handle toggling between edit and display modes

      // more code for display formatting
    });

This example highlights the scalability issues of the typical controller backed template examples you see. The more we add to a page, the larger the controller gets. In the end the complexity of a controller grows, make it more difficult to understand and maintain.

Not only does the controller grow, but the template also becomes very complex. We have 100s of lines of properties, computed properties and actions mixed with html markup. Breaking it into partials only makes it more difficult to follow what is the backing context of what template.

There are better ways in Ember to get the same functionality. We will later discuss how components and router states can help us break up this code.

### Issue 2: Handling State

Let's continue our example where we have a edit form for our contact model. On this page, we want to have the ability to delete the contact, but we first want to prompt the user to verify before actually deleting. This is done simple enough on our controller with the following code:

    App.ContactEditController = Em.Controller.extend({
      showConfirmDelete: false,

      actions: {
        toggleConfirmDelete: function() {
          this.toggleProperty('showConfirmDelete');
        },
        confirmDelete: function(model) {
         this.toggleProperty('showConfirmDelete');
          // logic to delete and navigate away
        }
      }
    });

And our template will have the following:

    {{#if showConfirmDelete}}
      Are you sure?
      <button {{action "corfirmDelete" model}}>Delete</button>
      <button {{action "toggleConfirmDelete"}}>Cancel</button>
    {{else}}
      <button {{action "toggleConfirmDelete"}}>Delete</button>
    {{/if}}

This works simply enough. We click delete, we are prompted to confirm, and if we do we delete. We make sure to clean up and reset the **showConfirmDelete** property on cancel or delete. But what happens if we click delete, and without canceling or confirming, navigate to another contact edit that uses the same model? That is where some unexpected state starts to creep in. The property **showConfirmDelete** would still be true, which means the user would already be prompted to confirm even though the model was completely different from when we first clicked delete.

This occurs because Ember Controllers are **singletons**. This is very important to how Ember functions because you cannot rely on the init part of lifecycle of a controller to reset it. The easy solution to this problem is to always set the property to the initial state in the Route's **setupController** hook like so:

    App.ContactEditRoute = Em.Route.extend({
      // some other stuff
      setupController: function(controller, model) {
        // other setup
        controller.set('showConfirmDelete', false);
      }
    });

This way when we navigate to another contact, we'll always reset to not show the confirm. While this is simple enough, there are a couple of problems. First, the actual setting up of state of our controller is done outside of the controller. Secondly, this is a trivial example, larger pages could have quite a bit of transient state in our controller we would have to reset.

Later we will look at how using a **Component** could better solve this problem.

### Issue 3: Proxies

Let's look at the following template snippet:

    <h2>{{name}}</h2>
    <h4>{{description}}</h4>

Now a quick question, are these properties on the model or the controller? That is the problem with **ObjectController** and to a lesser extent **ArrayController**, because they proxy model properties onto the controller, we can't differeniate the source of a property just by looking at the template.

I've found it to be best practice to always be explicit in templates and not rely on the proxies. Rather than the above, we can do something like:

    <h2>{{model.name}}</h2>
    <h4>{{description}}</h4>

By doing this, we can reliably know that the name property lives on the model and the description lives on the controller. Even so, this is simply a practice and that has to be enforced. Better yet is to avoid extending **Ember.ObjectController** and just extend **Ember.Controller**. This way, the property name would be undefined if you accidently wrote it the first way.

### Issue 4: Reuse / Template Disconnect

Another major concern with the use of controllers is the difficulty to reuse logic. While it is easy enough to reuse the controller logic itself through inheritance or mixins, the real difficulty is that we must also replicate the template itself. While this can be done through partials or the templateName property on a route, there is an inherant disconnect between controllers and templates. This makes testing more difficult and increases the likelyhood of bugs.

## Alternatives to Controllers

We have described in detail the shortcomings of Ember Controllers. A larger application that is heavily reliant on a lot of controller logic will eventually become a maintainence nightmare. For this reason, you should look to break out your controllers into other pieces. Ember gives us mechanisms that are superior to using controllers. Let's look at a few of our options.

### Use route definitions for your states

One of the best parts of Ember is it's url support. The easiest way to remove complexity of your application is to identify and break it apart into distinct states that are serialized into the url. This allows the framework to handle the setting up of state rather than tracking everything ourselves inside a controller. A perfect example is having the edit state separate from our detail state.

    // router.js
    this.resource('contact', function() {
      this.route('detail', { path: ':contact_id' });
      this.route('edit', { path: ':contact_id/edit' });
    });

Each of these states will have a route and controller responsible for just that state and will be much smaller than those needed to handle both.

### Break apart your UI into Components

For some parts of the application, you may end up with complicated UIs that have lots of transient state or calculation logic to determine what to display. Rather than putting this all in controllers and having it represented in the template through computed properties, we can instead use smaller components that represent a distinct portion of the UI.

This has several added benefits over controllers:

- Reuse. This is accomplished in two ways. First, a component is sandboxed so that all parameters must be passed in and actions must be explictly mapped to the outside world. This avoids collisions with actions of the same name. Secondly, we can compose components together. This allows us to use lower level components to build higher level components.

- There is a tighter connection between the logic and template representing a component, making it much easier to reason about.

- The templates end up being more of DSL than markup. This makes them easier to understand than property bindings and html everywhere.

Let's revisit the confirm delete we discussed earlier. Rather than fixing this up in the setupController hook, we could have just used a component like so:

    App.ConfirmDeleteComponent = Em.Component.extend({
      showCofirmDelete: false,

      actions: {
        toggleConfirmDelete: function() {
          this.toggleProperty('showConfirmDelete');
        },
        confirmDelete: function() {
         this.toggleProperty('showConfirmDelete');
         this.sendAction();
        }
      }
    });

We can move the existing portion of the template inside our component's template and put the following in it's place:

    {{confirm-delete action="confirmDelete"}}

By moving this to a component, we can rely on it being in the initial state when it is inserted into the DOM and each time we switch models. This means we don't need any extra logic to reset the state, it is done for us. Best of all, we could reuse this component throughout our application on other types of models.

Going back to the large controller in **Issue 1**, we can identify the sections of our template and break them up into components (omitting extra markup for clarity):

    {{contact-profile contact=model}}

    {{contact-activities contact=model activities=activities}}

    {{contact-groups contact=model groups=groups remove="removeGroup"}}

    {{contact-info contact=model}}

The components will contain the logic that was on the controller, along with the template needed. With a simple change like this, we have removed the majority of logic from the controller, simplified our template and made our application composable.

In my opinion, using components makes your UI much easier to understand and maintain than controllers + templates.

### Move out business logic

Another way to reduce logic in the controllers is to push business logic into models and objects that handle state. This helps separate UI logic from the business logic and makes things easier to reason about. A perfect example of this would be validation logic or auth managment. Often times this will be done through persistent models (like Ember Data models) or through objects that are injected using Ember's container.

## Guidelines for Using Controllers

Hopefully I've been convincing enough that controllers have a scalability issue and shown some alternatives you can use. This is not to say you should never use controllers, but you should know when they are doing to much and how to break them up into more maintainable blocks. Here are some guidelines I have to using controllers:

- Keep them as small and focused as possible.
- Avoid needs as much as possible. Cross controller dependencies makes the application brittle and difficult to reason about.
- Avoid too much inheritance. Try not to go more than a level deep and use mixins if possible.
- Never inherit a controller specific to one page for another controller. If that page's needs change, you may end up breaking the inheritting controller's page.
- Avoid business logic in the controller.
- Avoid managing state in controllers. Becuase they are singletons, you will find yourself in unexpected states that requires hacky code to fixup.

These constraints help keep controllers from overstepping their bounds and make your Ember application more understandable and maintainable.
