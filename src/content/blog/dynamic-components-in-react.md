---
title: "Dynamic Components in React"
description: "Recently, I've had the requirement at work of needing to build two apps off one React codebase. While the two apps would be very similar in structure and flow, there would be…"
pubDate: "2016-04-17T00:00:00"
---

Recently, I've had the requirement at work of needing to build two apps off one React codebase. While the two apps would be very similar in structure and flow, there would be several significant differences internally. Ideally, we wanted each app to ship only the code it needs, not the entire codebase.

The solution I came up with was to use React's [context](https://facebook.github.io/react/docs/context.html) feature. Although this feature is experimental, we can keep the surface area low and thus the risk can be minimal.

To keep the surface area touched by context small, we will follow the pattern used by libraries like react-redux and react-intl to create a provider and a function that creates higher order components with some set of values passed in as props.

Our provider component is minimalist:

```js
import { Component, Children, PropTypes } from "react";

export default class AppProvider extends Component {
  getChildContext() {
    return { config: this.props.config };
  }

  render() {
    return Children.only(this.props.children);
  }
}

AppProvider.childContextTypes = {
  config: PropTypes.object,
};
```

This provider expects to be passed in a config object which will provide app specific things like child components. We will get to this later.

And our helper function `injectConfig` is likewise small:

```js
import React from "react";

export default function injectConfig(WrappedComponent) {
  class ConnectedComponent extends React.Component {
    render() {
      return <WrappedComponent {...this.props} config={this.context.config} />;
    }
  }

  ConnectedComponent.contextTypes = {
    config: React.PropTypes.object,
  };

  return ConnectedComponent;
}
```

We can use this to wrap any components that need the dynamic config data.

The last piece of the puzzle is creating a config that takes in the app specific components. To handle this, we simply export an object with our data:

```js
import Login from "./components/Login";

export default {
  title: "App A",
  components: {
    Login: Login,
  },
};
```

This example config has an object with the app title and a login component.

Now that we have these pieces, we just need to wire them together.

```js
import React from 'react';
import ReactDOM from 'react-dom';
import AppProvider from '../components/AppProvider';
import App from '../components/App';
import config from './config';

ReactDOM.render(
  <AppProvider config={config}>
    <App>
  </AppProvider>
);
```

Now let's say somewhere within the `App` components nested children, we have a component `Main` that we want to have a `Login` component that is different for each app. We can make our `Main` component as so.

```js
import React from 'react';
import injectConfig from '../injectConfig';
import Header from'./Header';
import Footer from './Footer';

class Main extends React.Component {
  render() {
    const Login = this.props.config.components.Login;

    return (
      <Header />
      <Login />
      <Footer />
    );
  }
}

export default injectConfig(Main);
```

What we've done is mixed generic components like `Header` and `Footer` with our app specific component `Login`. This component doesn't need to know about anything specific to either of our apps, just that it will get passed a Login component via config.

To actually use `Main`, we can call just like any other component:

```js
import React from "react";
import Main from "./Main";

export default function App(props) {
  return (
    <div>
      <h1>Generic Header</h1>
      <Main />
    </div>
  );
}
```

We now have generic app structure but can create app specific components for each app where we need.
