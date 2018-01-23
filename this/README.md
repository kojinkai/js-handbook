# JS handbook

At the time of execution of every function, a property called `this` is set and it refers to the current execution context. `this` always refers to an object and which object depends on the context in which the function is being called. For example:

## Table of contents

- [this & functions](#this--functions)
- [this & objects](#this--objects)
- [this & constructors](#this--constructors)
- [this & setTimeout](#this--settimeout)
- [this & event handlers](#this--event-handlers)

## this & Functions

* In the global context or inside a function `this` refers to the window object.
* Inside IIFE (immediately invoked function expression) if you use "use strict", value of `this` is undefined.
```js
(function() {
  console.log(this) // Window
})()

(function() {
  'use strict';
  console.log(this) // undefined
})()
```

## this & Objects
* While executing a function in the context of an object, the object becomes the value of `this`
```js
const obj = {
  foo: function() {
    console.log('calling foo: ', this); // {foo: ƒ, bar: ƒ}
  }
}
```
## this & constructors
* If you use a constructor (by using the new keyword) to create an object, the value of `this` will refer to the newly created object.
```js
function Person(name, age) {
  this.name = name;
  this.age = age;
}

Person.prototype.sayName = function() {
  return this.name;
};

const billy = new Person('billy', 22);

billy.sayName() // billy

```
You can set the value of `this` to an object of your choosing by passing that object as the first argument of bind, call or apply
```js
const module = {
  x: 120,
  getX: function() {
    return this.x;
  }
};
module.getX() // 120

const boundThis = module.getX.bind({ x: 300 });

boundThis() // 300
```
## this & setTimeout
* Inside a setTimeout function, the value of `this` is the window object.
```js
const obj = {
  bar: function() {
    setTimeout(function() {
      console.log('calling bar: ', this); // Window
    }, 0)
  }
}
```

## this & event handlers
* For an event handler bound to a DOM element, value of `this` would be the element that the event was fired upon

Given the following HTML fragment
```html
<div class="element-container">Some content</div>

```

```js
// Logs the DOM element
document.querySelector('.element-container').addEventListener('click', function() {
  console.log(this) // <div class="element-container">Some content</div>
});
```
In many cases it is preferred to control the execution context of the callback handler. See the section on [binding a functions execution context](https://github.com/kojinkai/js-handbook/tree/master/functions#bind)

 
