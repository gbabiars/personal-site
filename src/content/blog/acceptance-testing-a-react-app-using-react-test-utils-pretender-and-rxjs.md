---
title: "Acceptance Testing a React App using React Test Utilities, Pretender and RxJS"
description: "As someone who is fairly new to building React apps, I got to the point where I needed to build acceptance tests for the app I'm working on to ensure that all the components…"
pubDate: "2016-02-12T00:00:00"
---

As someone who is fairly new to building React apps, I got to the point where I needed to build acceptance tests for the app I'm working on to ensure that all the components and modules are working together. What I wanted was something that was done in JavaScript and I could use the same test library as are used for unit tests. Having used Ember extensively in the past, I took inspiration from Ember's [acceptance test helpers](https://guides.emberjs.com/v2.3.0/testing/acceptance/). In the end, I was able to build out acceptance tests that were both clear and concise using [React Test Utilities](https://facebook.github.io/react/docs/test-utils.html), [Pretender](https://github.com/pretenderjs/pretender) and [RxJS](https://github.com/ReactiveX/RxJS).

The basic structure of a test is as follows:

- Pretender is used to mock ajax endpoints. This allows us to test different data scenarios including error states.
- React Test Utilities are used to simulate events on our DOM.
- RxJS is used to control the async flow of the test.

### Basic Example: Login

To demo how this might work, let's test a very basic scenario: logging in. Here is the code:

```js
import React from "react";
import { render, unmountComponentAtNode } from "react-dom";
import TestUtils from "react-addons-test-utils";
import Rx from "rxjs/Rx";
import Pretender from "pretender";

import init from "../init";

function waitFor(fn) {
  return Rx.Observable.interval(1)
    .filter(() => fn())
    .take(1);
}

export function ok(data) {
  return [200, { "Content-Type": "application/json" }, JSON.stringify(data)];
}

describe("login", function () {
  let root, server;

  beforeEach(function () {
    root = document.createElement("div");
    server = new Pretender();
  });

  afterEach(function () {
    unmountComponentAtNode(root);
    server.shutdown();
  });

  it("should render the welcome message on successful login", (done) => {
    server.post("https://server/authorize", () => ok("token"));

    init(root); // start our app rendering into root element
    root.querySelector("#username").value = "user";
    root.querySelector("#password").value = "pass";
    TestUtils.Simulate.submit(root.querySelector("form"));

    waitFor(() => root.querySelector("#welcome") !== null).subscribe(() => {
      expect(root.querySelector("#welcome").textContent).toEqual("Welcome!");
      done();
    });
  });
});
```

Let's break this down piece by piece to see how it works.

First, in our `beforeEach` we are setting up our root element that we will render into. We won't actually need to insert it into the DOM. We also create our Pretender server that will intercept xhr requests. The `afterEach` will ensure we tear down our app and stop our mock server.

In our spec, we mock out the /authorize endpoint so it will respond with a token.

The next section is where we will interact with our app. `init(root)` calls our app's init function and passes our root element for it to render into. It renders a login form by default. Next, we will fill in the username and password fields. Finally, we use React's Test Utils' Simulate helper to submit the form.

The last section is where we are going to do our assertions. Since the app has async code, we need to ensure that the async code has run (ajax request, promise resolution) and the UI has updated. To accomplish this, we are using the helper method:

```js
function waitFor(fn) {
  return Rx.Observable.interval(1)
    .filter(() => fn())
    .take(1);
}
```

The returned observable will poll every second checking if the input function returns a truthy value at which point it will stop checking. In our instance, we wait for the welcome message to be rendered which will happen after a successful login. Once the welcome message is present `subscribe` function will be called, where we will assert that we have the correct message, finally calling `done()` to let the framework know we have completed our async test.

### Complex Example: Saving a Record

While the previous example works well, it is pretty simplistic in that we only have one action (submitting the form). We could have easily accomplished this without observables. To see why RxJS is really powerful in this case, let's look at saving a record.

Our flow is as follows:

- The app will render a list of contacts with a new contact button.
- Clicking new will display the new contact form.
- Submitting the form will save the contact and display the contact detail.

```js
describe("create contact", function () {
  it("should save the contact and display the detail", (done) => {
    server.get("https://server/api/contacts", () => ok([]));
    server.post("https://server/api/contacts", (request) => {
      let contact = JSON.parse(request.requestBody);
      contact.id = "1";
      return created(contact);
    });

    init(root);

    waitFor(() => root.querySelectorAll(".contact-list-item").length > 0)
      .do(() =>
        TestUtils.Simulate.click(root.querySelector("#create-contact-btn")),
      )
      .mergeMap(() => waitFor(() => root.querySelector("#create-contact-form")))
      .do(() => {
        root.querySelector("#name").value = "John Doe";
        TestUtils.Simulate.submit(root.querySelector("#create-contact-form"));
      })
      .mergeMap(() => waitFor(() => root.querySelector("#contact-detail")))
      .subscribe(() => {
        expect(root.querySelector("#name").textContent).toEqual("John Doe");
        done();
      });
  });
});
```

Similar to the last example, let's go through this a section at a time. The first section mocks our server so that when we fetch all contacts we get an empty list and when we save a contact it is returned with an id.

We run `init(root)` the same as the last example. This time though we will assume the initial render is the contact list (token setup omitted).

The last section is where we have a lot more going on. The first `waitFor` checks for the the list to be rendered. Once it is, we'll use the `do` side effect to click the create contact button. Next, we'll run `mergeMap` and return a new observable from `waitFor` that checks for the form to be rendered. Next, we'll use `do` to fill in our create form and submit it. We'll use `mergeMap` again with another `waitFor`, this time checking that the detail has rendered. Finally, our `subscribe` block will run after everything has completed and we can run our assertions and call `done()`.

The code very much follows the outline of our test case defined above. The pattern is very easy to scale and repeat: do some action, wait for change. The best part is we haven't needed to write a bunch of extra boilerplate code, we're just using the built in RxJS operators (interval, filter, take, do, mergeMap).

### Conclusion

Overall, I've been very happy with the use of Pretender, React Test Utilities and RxJS for doing acceptance tests of my React application. These tools make it very easy to write clear tests that test the entire front end application. The biggest hurdle is learning observables and RxJS, but once you get a basic understanding on that it is pretty easy to follow.
