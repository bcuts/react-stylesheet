React Stylesheet
================

[![Travis build status](https://img.shields.io/travis/prometheusresearch/react-stylesheet/master.svg)](https://travis-ci.org/prometheusresearch/react-stylesheet)

React Stylesheet is a way to style React components with... React components!

## Installation

```
% npm install react-stylesheet
```

## Basic usage

The idea is that component should define a stylesheet to render its UI with.

A stylesheet is just a collection of React components. You ask what React
Stylesheet does for you then? It allows to define styled DOM components with
an easy API:

```javascript
import React from 'react'
import {createStylesheet} from 'react-stylesheet'

let stylesheet = createStylesheet({
  Root: { // generates <button /> with CSS class applied
    Component: 'button',
    fontSize: '12pt'
  },
  Caption: { // generates <button /> with CSS class applied
    Component: 'div',
    fontWeight: 'bold',
    hover: { // yes, pseudo classes are supported
      color: 'red'
    }
  }
})
```

Now we can define our component in terms of the stylesheet:

```javascript
class Button extends React.Component {

  render() {
    let {caption} = this.props
    let {Root, Icon} = stylesheet
    return (
      <Root>
        <Caption>{caption}</Caption>
      </Root>
    )
  }
}
```

## Stylable composite components

Sometimes you want to define a reusable component which you would want to style
later using different stylesheets.

You can define an initial stylesheet which does nothing but renders base DOM
components and then override that:

```javascript
import React from 'react'
import {attachStylesheet} from 'react-stylesheet'
import Icon from 'react-fa'

let stylesheet = {
  Root: 'button',
  Icon: Icon,
};

@attachStylesheet(stylesheet)
class Button extends React.Component {

  render() {
    let {caption, icon} = this.props
    let {Root, Icon} = this.props.stylesheet
    return (
      <Root>
        <Icon name={icon} />
        {caption}
      </Root>
    )
  }
}
```

What we did here is:

* We use `attachStylesheet` [higher order component][] to attach an initial
  stylesheet to a composite component.

* We use `stylesheet` prop passed to component to render component's UI.

Now the only part left is to produce a version of `<Button />` with different
styling. We use `styleComponent(Component, stylesheet)` function for that:

```javascript
import {styleComponent} from 'react-stylesheet'

let SuccessButton = styleComponent(Button, {
  Root: {
    color: 'white',
    backgroundColor: 'green',
    hover: {
      color: 'red'
    }
  },
  Icon() {
    return <Icon name="ok" />
  }
})
```

We pass `styleComponent()` a stylesheet which is merged into the original one:

* If you pass a component (`Icon` in the example above) it is used instead of
  the original one.

* If you pass an object:

  * If it overrides DOM component in the original stylesheet, then the object is
    treated as a set of CSS styles. It's compiled to CSS class and a new styled
    DOM component wrapper is generated with the CSS class attached.

  * If it overrides composite component in the original stylesheet, then it's
    being used to style this component with recursive `styleComponent` function
    call.

The last point allows to style component hierarchies with easy:

```javascript
let StyledForm = styleComponent(Form, {
  Root: {
    ...
  },
  // this is the same as calling styleComponent(SubmitButton, { ... })
  SubmitButton: {
    Root: {
      ...
    },
    Icon: {
      ...
    }
  }
})
```

## Styled DOM components

You can also produce styled DOM components with `styleComponent` function:

```javascript
import {styleComponent} from 'react-stylesheet'

let StyledDiv = styleComponent('div', {
  color: 'red'
})

React.render(<StyledDiv />, ...)
```

That results in:

```html
<style>
  .autogenerated_class_name {
    color: red;
  }
</style>
...
<div class="autogenerated_class_name" />
```

The `<style>` element is attached only at least one of the corresponding styled
DOM components are mounted into the DOM.

## States for styled DOM components

Styled DOM components are allowed to have variants:

```javascript
import {styleComponent} from 'react-stylesheet'

let StyledDiv = styleComponent('div', {
  color: 'red',
  hover: {
    color: 'black'
  },
  customState: {
    backgroundColor: 'red'
  }
})
```

States like `hover`, `active`, ... are compiled into pseudoclasses and into
regular classes as well.

You can toggle variants by passing variant names as keys to `variant` prop of styled
DOM components:

```javascript
<StyledDiv variant={{hover: false, active: true}} />
```

## Helpers for DOM stylesheets

React Stylesheet provides helpers to define DOM stylesheets:

```javascript
import {styleComponent} from 'react-stylesheet'
import {rgba, padding, none} from 'react-stylesheet/css'

let Warning = styleComponent('div', {
  color: rgba(245, 123, 12),
  padding: padding(10, 20),
  textDecoration: none,
})
```

## Credits

React Stylesheet is free software created by [Prometheus Research][] and is
released under the MIT license.

[Prometheus Research]: http://prometheusresearch.com
[higher order component]: https://gist.github.com/sebmarkbage/ef0bf1f338a7182b6775
[react-fa]: https://github.com/andreypopp/react-fa
