---
title: "AngularJS Patterns in React"
description: "When switching from one technology to another, it is common for developers to still think in the original technology they are familiar with. Currently, my team is working on…"
pubDate: "2018-04-29T00:00:00"
---

When switching from one technology to another, it is common for developers to still think in the original technology they are familiar with. Currently, my team is working on migrating from an AngularJS (aka angular 1) to React.js. One of the problems we have to solve is how to replicate existing filters, directives and components, preferably with the same markup as the current output to avoid breaking CSS. In React, the solution to all of these are Components, but it is the different patterns we use with those components that will allow us to accomplish this. In this post, we explore some common use cases in AngularJS and how we can replicate the functionality in React, using the latest features in React 16.3.

## Filters

Let's start by looking at filters. In AngularJS, filters allow us to pipe in data, such as text, and the filter will transform the output. This is commonly used for things like currency formatting or translations.

### Example: Text Formatting

Here we'll look at a common case, which is formating text. For this, well recreate a filter that converts a string to lowercase.

```js
// angular lowercase filter
const lowercase = () => (text) => text.toLowerCase();

// in our template
{
  {
    "My Text" | lowercase;
  }
}
```

To solve this in React, we can take advantage of the fact that as of React 16, [components can return strings](https://reactjs.org/blog/2017/09/26/react-v16.0.html#new-render-return-types-fragments-and-strings) without being wrapped in spans or other elements.

```js
const Lowercase = ({ text }) => text.toLowerCase();

<Lowercase text="My Text" />;
```

### Example: Translations

In some cases, we need to have injections into our filter to use for formatting. Here we'll look at a common case of translation strings.

```js
// angular translate filter with injection of translations factory
const translate = (translations) => {
  return value => translations.translate(value);
};

// in our template
<h1>{{ 'Hello World' | translate }}</h1>
```

We can recreate this in React by taking advantage of the new [context api](https://reactjs.org/docs/context.html) in order to access the translate function from a child component without passing it through our component hierarchy.

```js
// create our context
const TranslateContext = React.createContext(() => '');

// create a component that consumes the context
const Translate = ({ value }) = (
  <TranslateContext.Consumer>
   {translate => translate(value)}
  </TranslateContext.Consumer>
);

// create a component that utilizes our Translate component
const HelloWorld = () => <h1><Translate value="Hello World" /></h1>

// render our component that has Translate as a child of our TranslateContext
ReactDOM.render(
  <TranslateContext.Provider value={translate}>
    <HelloWorld />
  </TranslateContext.Provider>,
  el
);
```

Let's take a moment to go over what is happening. First, we create our TranslateContext, which takes in a default value as a function always returning an empty string. Next, we'll create a Translate component which utilizes the [function as a child](https://reactjs.org/docs/render-props.html#using-props-other-than-render) pattern. The translate function from our context is passed as the argument, which we then call with the value passed in to props of our Translate component. Next, we'll create a HelloWorld component which consumes our Translate. Finally, we'll render our HelloWorld as a child of TranslateContext.Provider, which takes in the actual translate function that will be used by our child consumers, overriding the default value.

While this may seem like a lot, remember that for most apps, all of this set up will happen once at the top level of the app.

## Directives

Directives in AngularJS allow a variety of benefits, from composition of functionality to modifying the content of an element. I'm going to focus only on attribute directives, since element level are just components, which I'll cover later.

### Example: SVG icons

In AngularJS, attribute directives are often used to allow the use of native elements but still modify the class attribute and add some inner elements. This is required because components will create a custom element, which can mess with CSS layout and give up control of adding things like ids. An example of this is an svg icon where we want to have an svg element, but modify the inner markup.

```js
const svgIcon = () => ({
  controller: function ($element) {
    this.$onInit = () => {
      $element.addClass("svg-icon");
    };

    this.$onChanges = () => {
      $element
        .empty()
        .append(`<use xlink:href="/assets/sprite.svg#${name}"></use>`)
        .addClass(`svg-icon--${name}`);
    };
  },
  controllerAs: "$ctrl",
  bindToController: {
    name: "<svgIcon",
  },
  restrict: "A",
  scope: {},
});

// in our template
<svg svg-icon="'arrow-up'"></svg>;
```

What we are doing here is taking in the name and combining it with our sprite url, then modifying the inner html. We're also adding some css to the root element.

In React, we can just use a component for this since we get to control what the root element is going to be, rather than being forced into a custom element.

```js
const SvgIcon = ({ name }) => (
  <svg className={`svg-icon svg-icon--${name}`}>
    <use xlinkHref={`/assets/sprite.svg#${name}`}></use>
  </svg>
);

<SvgIcon name="arrow-up">
```

### Example: Theme Styles

Another usage of directives can be to modify the styling of an element based for theme specific styles. Where we gave up some control of the element in the last example by assuming it was an SVG element, we need more flexibility for this functionality.

```js
const themeClass = () => ({
  controller: function ($element, themeConfig) {
    this.$onInit = () => {
      $element.addClass($element.attr(`brand-class-${themeConfig}`));
    };
  },
});

<button
  type="button"
  class="button"
  theme-class
  theme-class-foo="button--inverse"
  theme-class-bar="button--primary"
>
  Click Me!
</button>;
```

What is happening here is we have some global config injected into the directive to let us know which theme we're using. Let's assume we have 3 themes: foo, bar and baz. In our usage, we have common classes put on the element and then theme specific classes that will be applied only for that theme. So if we're on foo, it would render "button button--inverse", bar would render "button button--primary" and baz would just render "button".

To recreate this in React, we can take advantage of function as a child and the new context api and create a component that rather than renders anything itself, just calls children with some computed data that it can use itself.

```js
const ThemeContext = React.createContext({ name: "foo" });

const ThemeClass = (props) => (
  <ThemeContext.Consumer>
    {(theme) => props.children(props[theme.name] || "")}
  </ThemeContext.Consumer>
);

const App = () => (
  <div>
    <h1>My App</h1>
    <div>
      <BrandClass foo="button--inverse" bar="button--primary">
        {(themeClass) => (
          <button type="button" className={`button ${themeClass}`}>
            Click Me!
          </button>
        )}
      </BrandClass>
    </div>
  </div>
);

ReactDOM.render(
  <ThemeContext.Provider value={{ name: "bar" }}>
    <App />
  </ThemeContext.Provider>,
  el,
);
```

Let's go over what is happening in this example. First we are creating our context with a default foo. Next, we create our ThemeClass component which takes in props which will be the theme specific styles (this is dynamic). We'll use the context callback to get the theme with the name, which we can use to look up the theme specific prop, if any, is present and call children with this brand specific style passed to it. In the usage in App, we can then use themeClass in our callback to populate the button with custom styles. The benefit of this pattern is that we can use it anywhere to do anything from theme specific classes to attributes and on.

## Components

With React, just about everything is a component. While AngularJS has components, the mechanics are quite different, even though the data flow is somewhat similar. While I don't want to go to in depth on every React feature, I do want to touch on some patterns of AngularJS components that can easily be replicated and improved on in React.

### Example: Basic with Conditionals

The first case I want to discuss is conditional logic to hide or show pieces of a component. In AngularJS, this is usually done via template using ng-hide, ng-show or ng-if. With React, trying to do this type of mechanism in JSX is pretty messy, which is why we tend to decompose components down to smaller sub-components (it is also much easier in React because we don't have to declare everything globally). To highlight this, let's take an example of a detail section we can toggle to show or hide via a button.

```js
const template = `
<div>
  <button type="button" ng-click="$ctrl.toggleDetails">
    Toggle
  </button>
</div>
<div ng-if="$ctrl.isDetailVisible">
  <h3>Details</h3>
  <div>
    More Info here
  </div>
</div>
`;
const demo = {
  template,
  controller: function () {
    this.isDetailVisible = false;

    this.toggleDetails = () => {
      this.isDetailVisible = !this.isDetailVisible;
    };
  },
};
```

Here our AngularJS component encapsulates the state and updates the isDetailVisible property, which is used as the conditional in the template. Let's compare that with a React equivalent:

```js
const Details = () => (
  <div>
    <h3>Details</h3>
    <div>More Info here</div>
  </div>
);

class Demo extends React.Component {
  constructor(props) {
    super(props);
    this.state = { isDetailVisible: false };
    this.onToggle = this.onToggle.bind(this);
  }

  onToggle() {
    this.setState((prevState) => ({
      isDetailVisible: !prevState.isDetailVisible,
    }));
  }

  render() {
    const { isDetailVisible } = this.state;
    return (
      <div>
        <div>
          <button type="button" onClick={this.onToggle}>
            Toggle
          </button>
        </div>
        {isDetailVisible ? <Details /> : null}
      </div>
    );
  }
}
```

While we could inline Details, I've moved it out for separation. This is more code than the AngularJS equivalent, the benefit is that our Details is completely separate from the container, which allows us to focus on the specifics of that component separate from the container. Unlike AngularJS, adding the component doesn't necessarily add an extra layer to the DOM since our top level can be a div (or Fragment in React 16+). In the end, template vs. small components is a subjective trade off that some will like and some won't.

### Example: Recreating Slots/Transclusion

Another common usage in components is the use of slots to build container components that intersperse markup from the container with spaces for parent component to control directly certain parts of the markup. To highlight this, we'll create a layout component for a details card that has header and body slots.

```js
const template = `
<div class="details-card">
  <div class="details-card__header" ng-transclude="header"></div>
  <div class="details-card__body" ng-transclude="body"></div>
</div>
`;
const detailsCard = {
  template,
  transclude: {
    header: '?detailsCardHeader',
    body: '?detailsCardBody'
  }
};

// usage
<details-card>
  <details-card-header>
    Header Text
  </details-card-header>
  <details-card-body>
    <div>
      Body Content
    </div>
  </details-card-body>
</div>
```

Personally, I've never been able to remember this syntax. There is a lot of ceremony to set it up. With React, there are several ways to accomplish the same depending on what type of api we want the component to have. For this example, we're going to take in the header and body as props.

```js
const DetailsCard = ({ header, body }) => (
  <div className="details-card">
    <div className="details-card__header">
      {header}
    </div>
    <div className="details-card__body">
      {body}
    </div>
  </div>
);

// usage
const Header = () => 'Header Text';
const Body = () => (
  <div>
    Body Content
  </div>
);

ReactDOM.render(
  <DetailsCard header={<Header />} body={<Body />} />
  </DetailsCard>,
  el
);
```

This is a much simpler pattern than the AngularJS equivalent and has the added benefit of not polluting the markup with a bunch of extra custom elements. For more complex cases, we can do things that are not really possible in AngularJS such as passing computed data on the container to the children via [render props](https://reactjs.org/docs/render-props.html) or function as a child.

### Example: UI Router Link

The final example I want to give is a fairly specific case that has come up, but highlights the flexibility of React's component api. When migrating from AngularJS to React, there came a need to have an interop between UI router and the parts built in React. With a combination of the new context api and a basic component, it is possible to recreate the ui-state directive for UI router.

```js
const StateContext = React.createContext({});
const StateLink = ({ name, children }) => (
  <StateContext.Consumer>
    {state => (
      <a href={state.get(name).url}
         onClick={
           e => {
             e.preventDefault();
             state.go(name);
           }
         }>
        {children}
      </a>
    )}
  </StateContext.Consumer>
);
const Navigation = () => (
  <div>
    <StateLink name="foo">
    <StateLink name="bar">
  </div>
);

// in an angular component
const navigation = {
  controller: ($element, $state) {
    this.$onInit = () => {
      ReactDOM.render(
        <StateContext.Provider value={$state}>
          <Navigation />
        </StateContext.Provider>,
        $element[0]
      );
    };
  }
};
```

What is happening here is we are using context to access UI router's state service. The benefit of this pattern is we don't have to pollute our React components with angular specific props, so when we swap out to something like React Router, we would not have to change the public api of our Navigation component.

## Conclusion

Hopefully this post highlights how React's api makes it possible to recreate much of the AngularJS api. The biggest difference is that we've been able to recreate these different pieces using just components, the context api and a few patterns. This simplicity is extremely powerful and highlights why React is such a great tool for building UIs.
