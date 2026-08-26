---
title: "Ember Routing - Passing Context vs Parameter"
description: "When linking and transitioning in an Ember application, there are two ways to pass context for a route with a dynamic parameter: pass a context object or pass the parameter…"
pubDate: "2015-10-05T00:00:00"
---

When linking and transitioning in an Ember application, there are two ways to pass context for a route with a dynamic parameter: pass a context object or pass the parameter value itself. In the past, I have mostly done the prior, but recently have begun using the latter more. To understand why we might want to use a parameter over the model, let's explore a little example.

Let's take a look at the canonical blog post example. Well have a router with a post route:

```js
this.route("posts", function () {
  this.route("post", { path: "/:post_id" });
});
```

And a post model:

```js
DS.Model.extend({
  title: DS.attr(),
  body: DS.attr(),
});
```

Then in our template we're going to link from the list to the detail and pass our post model.

```html
{{#each model as |post|}} {{link-to post.title 'posts.post' post}} {{/each}}
```

Now let's throw a curve at this with a more complex example. Let's say our posts are fairly large and we don't want to load all the data up front. To speed things up, we only bring back the minimum data we need to render the list:

```json
{
  "posts": [
    {
      "id": "1",
      "title": "My Fist Post"
    }
  ]
}
```

The problem here is that we no longer will have the title loaded for our post. So when we navigate to our detail, we will have to load our extra data.

Before solving this problem, let's revisit what happens when you transition or hit a route directly by url. When transitioning without a context or hit the url directly (or refresh), we follow the path of `model -> setupController -> renderTemplate` (ignoring the before, after, etc). However, when we pass a dynamic context via `link-to` or `transitionTo`, we actually end up with `setupController -> renderTemplate`, skipping the model hook. This is often confusing to beginners, but also makes solving problems like ours more difficult.

Let's get back to our problem at hand. We've decided that we can just use the `setupController` hook to reload our model.

```js
setupController(controller, model) {
  controller.set('model', model);
  model.reload();
}
```

This will solve our problem, but we've potentially created a bad user experience. To understand why, we need to understand the mechanics of the different route hooks. `model` is a hook that fetches data, continuing on immediately if the return value is not a promise or pausing until fulfillment if the return value is a promise. This allows us to block and display a loading state if we need. `setupController` on the other hand does not ever pause. In our case this means that `renderTemplate` can be executed before `model.reload` completes. This will cause our post to not have a body at first, then when `reload` completes, our body will be displayed. This flicker can be annoying to the user and it also means we need some feedback to the user that the body is loading like:

```html
{{#if model.body}} {{model.body}} {{else}} Loading Post... {{/if}}
```

Given that this feels pretty hacky, let's find another way. Instead of passing the post model directly from our list, let's update our list template and instead just pass the post id.

```html
{{#each model as |post|}} {{link-to post.title 'posts.post' post.id}} {{/each}}
```

We still match the route parameters, but because we are now passing a primitive value, we will actually hit the hooks as `model -> setupController -> renderTemplate`. This gives us consistency with doing a refresh. Now we can override the default model hook and do the side loading in there:

```js
model(params) {
  return this.store.find('post', params.post_id).then(post => {
    // post was already loaded but did not have body
    if(!post.get('body')) {
      return post.reload();
    }
    // if post has body, just return the post
    return post;
  });
}
```

This will give us a nice consistent behavior. No matter how we hit the post route (link, refresh, etc), we will always hit the model hook and the resolved value from that hook will always have the `post` attribute populated. If the load takes long or fails, we will transition into our loading and error states respectively so we will have consistentcy there. We no longer need conditional logic in our template, we can just assume the data is fully loaded and render.

To recap, passing the parameter instead of the context has several benefits:

- Consistency in which hooks get called between the different ways you can enter a route.
- Eliminates the need to worry about the serialization of context objects to the url parameter.
- By hitting the model hook we can block rendering when we need to load additional data and get the loading/error states without extra work.
- With proper caching and/or Ember Data, the overhead from loading the item from cache via id is negligible.
- Can still load dynamic information in the `setupController` hook if need be.

There can be some potential drawbacks:

- If we are not using a cache, hitting the model hook every time will cause a lot of extra ajax calls and hurt percieved performance.

Given that it adds consistency to the lifecycle and makes it easier to solve problems like side-loading additional data, passing a dynamic parameter via `link-to` or `transitionTo` is becoming more appealing than passing a context object.
