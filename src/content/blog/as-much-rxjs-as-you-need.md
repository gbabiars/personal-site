---
title: "As Much RxJS as You Need"
description: "Note: Examples in this post use RxJS 5"
pubDate: "2016-02-21T00:00:00"
---

_**Note: Examples in this post use RxJS 5**_

Recently I have been using [RxJS](https://github.com/ReactiveX/RxJS) in work apps, starting with using it to coordinate async tests and recently replacing complex data flows. I have also started playing with it for non-work stuff including some with [Cycle.js](http://cycle.js.org/). The ability to use observables a little or fully buy in has led me to realize one of the main reasons I like RxJS: you can use as much as you need!

Being able to use just the parts you need is very nice since it allows integration with other libraries and frameworks. Since the observable is such a powerful primitive, it also allows the building of frameworks like Cycle.js where everything is an observable.

###Just a little

RxJS can be very approachable as someone new because you can introduce it slowly to your project. A simple example of this would be if we wanted to use observables instead of promises for our data layer. Here is a simple example:

```js
import Rx from "rxjs/Rx";
import "rxjs/add/observable/dom/ajax";

export default function get(url) {
  return Rx.Observable.ajax.get(url).toPromise();
}
```

The `toPromise` operator converts our observable to a promise so we can keep our existing promise based api. Let's say we wanted to add some retry in case we get random network errors:

```js
import Rx from "rxjs/Rx";
import "rxjs/add/observable/dom/ajax";

export default function get(url) {
  return Rx.Observable.ajax.get(url).retry(2).toPromise();
}
```

This code simply says if we get an error, retry 2 times and fail error out after that. We could also use the retryWhen operator to have more complex retry logic:

```js
import Rx from "rxjs/Rx";
import "rxjs/add/observable/dom/ajax";

export default function get(url) {
  return Rx.Observable.ajax
    .get(url)
    .retryWhen((errors$) => {
      // more complex retry logic
    })
    .toPromise();
}
```

With this we can do things like refresh an auth token when we receive a 401 and then retry the previous request after. What makes RxJS so appealing is that the data flow is followable even with complex things like retry logic. The best part is that we don't need to change the exposed api.

If we ever did want to change our consumer code to use observables, we simply remove `toPromise()` from the `get` function and change our consumer code as such:

```js
// before
get(url).then(
  (data) => {
    // success
  },
  (err) => {
    // error
  },
);

// after
get(url).subscribe(
  (data) => {
    // success
  },
  (err) => {
    // error
  },
);
```

### Interoperability

When using a new library, it is very important how it will play with other frameworks and libraries you use. RxJS gives us many helpers out of the box that allow us to interop with other ways of doing asynchorous programming in JavaScript, mainly promises and callbacks.

As we saw in the previous section the [toPromise](https://github.com/ReactiveX/RxJS/blob/master/spec/operators/toPromise-spec.js) operator conveniently converts our observable to a promise. The operator [fromPromise](https://github.com/ReactiveX/RxJS/blob/master/spec/observables/from-promise-spec.js) is also provided which takes in a promise and creates an observable. This is very useful if you want to consume something like jQuery's ajax promise or some other library that returns promises.

RxJS provides two helpers for dealing with callbacks: [bindCallback](https://github.com/ReactiveX/RxJS/blob/master/spec/observables/bindCallback-spec.js) and [bindNodeCallback](https://github.com/ReactiveX/RxJS/blob/master/spec/observables/bindNodeCallback-spec.js). These allow you to wrap functions and return an observable representing the result of the function.

### Taking it further

While you may want to just introduce a little RxJS to your codebase, you may end up wanting to use it more and more. Luckily there are many projects that are starting to take RxJS as a dependency. [Angular 2](https://angular.io/) uses RxJS for many internal pieces. If you use React, [rx-react](https://github.com/fdecampredon/rx-react) and [thisless React](https://github.com/jas-chen/thisless-react) allow you to observables for state and actions. Lastly, [Cycle.js](http://cycle.js.org/) is a small framework built using RxJS observables to define the entire flow of your JS application.

### Taking it for a spin

The RxJS team has done an amazing job of providing a primitive and utilities that can interop with other libraries and frameworks in the JS space. Because of this, adopting it is a very low commitment. If you haven't already tried it out, I highly recommend giving it a chance.
