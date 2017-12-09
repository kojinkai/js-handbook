# JS Gotchas

## Table of contents

- [Closures and SetTimout](#closures--settimeout)
- [Undefined vs NULL](#undefined-vs-null)
- [Variable Hoisting](#variable-hoisting)
- [The this keyword](#the-this-keyword)
- [Constructing Primitive Types](#constructing-primitive-types)

## Closures & SetTimeout

```js
// Q: What will the following code output?
const arr = [10, 12, 15, 21];
for (var i = 0; i < arr.length; i++) {
	setTimeout(function() {
		console.log(`Index ${i} element: ${arr[i]}`)
	}, 3000);
}

// A: Index 4 element: undefined (4 times)
```

### Discussion
The problem deals with closures, setTimeout and scope. The setTimeout method takes in a function which creates a *closure*. The closure has access to variable `i` in the outer scope. After the timeout of 3 seconds has elapsed the callback is executed and prints the value of `i` by which time the has ended and the value of `i` is now 4 and `arr[4]` does not exist and is therefore `undefined`.

### Solutions
There are two solution to this problem. The first involves creating a parametrised inner function and passing the current value of `i` within each loop iteration.
```js
const arr = [10, 12, 15, 21];
for (var i = 0; i < arr.length; i++) {
  // pass in variable i so the innermost function
  // has access to the value of i in a given iteration of the loop
  // in scope
  setTimeout(function(i_local) {
    return function() {
      console.log(`Index ${i_local} element: ${arr[i_local]}`)
        }
  }(i), 3000);
}
```
The second possible solution uses ES6 and is more concise
```js
const arr = [10, 12, 15, 21];
for (let i = 0; i < arr.length; i++) {
  // The ES5+ let syntax creates a new binding
  // every time the function is called
  setTimeout(function() {
    console.log(`Index ${i} element: ${arr[i]}`)
  }, 3000);
}
```

## Undefined vs NULL
```js
// Q: What will the following code evaluate to?
typeof null // A: object
typeof undefined // A: "undefined"
```
### Discussion
> You may wonder why the typeof operator returns 'object' for a value that is null. This was actually an error in the original JavaScript implementation that was then copied in ECMAScript. Today, it is rationalized that null is considered a placeholder for an object, even though, technically, it is a primitive value.
> undefined means a variable has been declared but has not yet been assigned a value whereas null is an assignment value. It can be assigned to a variable as a representation of no value.
> -- <cite>Professional JS For Web Developers (Wrox)</cite>

Also, undefined and null are two distinct types: undefined is a type itself (undefined) while null is an object.
Unassigned variables are initialized to undefined. JavaScript never sets a value to null. That must be done programmatically.

## Variable Hoisting
```js
// Q: What will happen when this function is executed?
function doSomething() {
  console.log(bar);
  var bar = 111;
}
// A: doSomething() // logs undefined
```
### Discussion
This issue deals with what is known as "hoisting" in JavaScript.
Variable declarations (and declarations in general) are processed before any code is executed and so declaring a variable anywhere in the code is equivalent to declaring it at the top.
This means that a variable can appear to be used before it's declared as though the variable declaration was moved to the top of the function or global code.

Therefore the above `doSomething()` function is implicitly understood as
```js
function doSomething() {
  var bar;
  console.log(bar); // undefined
  bar = 111;
  console.log(bar); // 111
}
```

Consider also the following example with undeclared variables:
```js
function doSomethingElse() {
  console.log(foo);
  console.log(bar);
  var bar = 111;
}

doSomethingElse() // Uncaught ReferenceError: foo is not defined
```
In this instance bar is not declared as a variable so an Uncaught ReferenceError is thrown when bar is encountered. However the execution context knows about any declarations before a function is ran and will initialise these variables to undefined.

### Solution
Declare variables at the top of the enclosing scope

## The this keyword
Q: Explain this

Answer: At the time of execution of every function, a property called `this` is set and it refers to the current execution context. `this` always refers to an object and which object depends on the context in which the function is being called. For example:

In the global context or inside a function `this` refers to the window object.
Inside IIFE (immediate invoking function) if you use "use strict", value of `this` is undefined.
```js
(function() {
  console.log(this) // Window
})()

(function() {
  'use strict';
  console.log(this) // undefined
})()
```
*To access the window object inside IIFE with "use strict", you would have to pass `window` to the IIFE as an argument.*
While executing a function in the context of an object, the object becomes the value of `this`
Inside a setTimeout function, the value of `this` is the window object.
```js
const obj = {
  foo: function() {
    console.log('calling foo: ', this); // {foo: ƒ, bar: ƒ}
  },
  bar: function() {
    setTimeout(function() {
      console.log('calling bar: ', this); // Window
    }, 0)
  }
}
```
If you use a constructor (by using new keyword) to create an object, the value of `this` will refer to the newly created object.
You can set the value of `this` to any arbitrary object by passing the object as the first parameter of bind, call or apply
For dom event handler, value of `this` would be the element that fired the event

## Constructing Primitive Types
```js
Q: what do the following logs output?
const emptyConstructedString = new String('');
const emptyStringLiteral = '';
console.log(typeof emptyConstructedString);
console.log(typeof emptyStringLiteral);

// A: 
console.log(typeof emptyConstructedString); // "object"
console.log(typeof emptyStringLiteral); // "string"
```
### Discussion
Although we are passing empty string to the string constructor, calling `new String()` will naturally create a new object instance. The same is true when calling `new Boolean()`