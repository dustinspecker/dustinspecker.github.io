---
layout: post
title: Creating a JSX Pragma
---

JSX is a neat syntax, but can cause confusion to users that aren't quite familiar with how the JSX transformer works.

Babel converts JSX such as the following:

```javascript
import React from 'react'

export default props =>
  <div title='hi'>
    <ul>
      {props.items.map(item =>
        <li>{item.name}</li>
      )}
    </ul>
  </div>
```

to:

```javascript
import React from 'react'

export default props =>
  React.createElement(
    'div',
    {title: 'hi'},
    React.createElement(
      'ul',
      null,
      props.items.map(item =>
        React.createElement(
          'li',
          null,
          item.name
        )
      )
    )
  )
```

The JSX transformer convert JSX into function calls. This function is the JSX pragma. A JSX pragma accepts a tag name, an attributes object, and the rest of the arguments passed are the element's children.

With this in mind, we can easily create our own JSX pragma.

First, let's handle only the tag name. We'll create a function accepting a tag name that returns an element of that type.

```javascript
const docCreateElement = tag => {
  const element = document.createElement(tag)

  return element
}
```

Next, let's handle accepting an attributes object.

```javascript
const docCreateElement = (tag, attrs) => {
  const element = document.createElement(tag)

  Object.keys(attrs).forEach(attr => {
    element.setAttribute(attr, attrs[attr])
  })

  return element
}
```

Next, let's handle attaching event handlers via the attrs object.

```javascript
const docCreateElement = (tag, attrs) => {
  const element = document.createElement(tag)

  Object.keys(attrs).forEach(attr => {
    if (typeof attrs[attr] === 'function') {
      // convert `onClick`, etc. to `click`
      const eventName = attr.slice(2).toLowerCase()
      element.addEventListener(eventName, attrs[attr])
    } else {
      element.setAttribute(attr, attrs[attr])
    }
  })

  return element
}
```

Next, let's handle appending children to the created element.

```javascript
const docCreateElement = (tag, attrs, ...children) => {
  const element = document.createElement(tag)

  Object.keys(attrs).forEach(attr => {
    if (typeof attrs[attr] === 'function') {
      // convert `onClick`, etc. to `click`
      const eventName = attr.slice(2).toLowerCase()
      element.addEventListener(eventName, attrs[attr])
    } else {
      element.setAttribute(attr, attrs[attr])
    }
  })

  children.forEach(child => {
    if (typeof child === 'string') {
      element.appendChild(document.createTextNode(child))
    } else {
      element.appendChild(child)
    }
  })

  return element
}
```
