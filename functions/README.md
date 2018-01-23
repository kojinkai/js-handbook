# Functions
Functions are a really important topic in JavaScript.

## Table of contents

- [bind](#bind)

## Bind
`.bind()` is a function defined natively on the Function object prototype. `.bind()` is called with a context and when called returns a function whose `this` keyword is set to the provided context value you pass in.

Consider the following example
```js
const module = {
  x: 120,
  getX: function() {
    return this.x;
  }
};
module.getX() // 120
const retrieveX = module.getX;
retrieveX(); // undefined (The function gets invoked within the global scope

const boundGetX = retrieveX.bind(module);
boundGetX(); // 120
```

### Discussion
Using `.bind()` can be useful for _partially applying_ functions with a context. Put another way, if a function, such as `module.getX` shown above, references its `this` property, we can use bind to pre-prepare `getX` with a new `this` value therefore influencing the outcome of the function call. Since `.bind()` returns a function, we can generate a function with a this value store within.

```js
const module = {
  x: 120,
  getX: function() {
    return this.x;
  }
};

const partiallyAppliedWithANewContext = module.getX.bind({ x: 300 });

partiallyAppliedWithANewContext() // 300
```

With this in mind `.bind()` is also useful for controlling the context when attaching event handlers to DOM elements. Since the `this` keyword is automatically bound to the DOM element the event was called upon, it is often useful to attach a callback function bound to another context, a controller class for example, in order that DOM events can be used to interact with properties and methods in other scopes.

Given the following DOM element

```html
<div class="element-container">Some content</div>

```

```js
// Logs the DOM element
document.querySelector('.element-container').addEventListener('click', function() {
  console.log(this) // <div class="element-container">Some content</div>
});


// Logs current context
document.querySelector('.element-container').addEventListener('click', function() {
  console.log(this);
}.bind(this));

// Logs current context
document.querySelector('.element-container').addEventListener('click', () => {
  console.log(this)
});

```

See also the section on the [this keyword](https://github.com/kojinkai/js-handbook/tree/master/this)
