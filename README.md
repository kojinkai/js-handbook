# JS Gotchas

## Table of contents

- [Closures and SetTimout](#closures--settimeout)
- [Undefined vs NULL](#undefined-vs-null)
- [Variable Hoisting](#variable-hoisting)
- [The this keyword](#the-this-keyword)
- [Constructing Primitive Types](#constructing-primitive-types)
- [Equality Checking](#equality-checking)

## Closures & SetTimeout

```js
// Question: What will the following code output?
const arr = [10, 12, 15, 21];
for (var i = 0; i < arr.length; i++) {
	setTimeout(function() {
		console.log(`Index ${i} element: ${arr[i]}`)
	}, 3000);
}

// Answer: Index 4 element: undefined (4 times)
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
// Question: What will the following code evaluate to?
typeof null // Answer: object
typeof undefined // Answer: "undefined"
```
### Discussion
> You may wonder why the typeof operator returns 'object' for a value that is null. This was actually an error in the original JavaScript implementation that was then copied in ECMAScript. Today, it is rationalized that null is considered a placeholder for an object, even though, technically, it is a primitive value.
> undefined means a variable has been declared but has not yet been assigned a value whereas null is an assignment value. It can be assigned to a variable as a representation of no value.
> -- <cite>Professional JS For Web Developers (Wrox)</cite>

Also, undefined and null are two distinct types: undefined is a type itself (undefined) while null is an object.
Unassigned variables are initialized to undefined. JavaScript never sets a value to null. That must be done programmatically.

## Variable Hoisting
```js
// Question: What will happen when this function is executed?
function doSomething() {
  console.log(bar);
  var bar = 111;
}
// Answer:
doSomething() // logs undefined
```
### Discussion
This issue deals with what is known as "hoisting" in JavaScript.
Variable declarations using `var` are processed before any code is executed and so declaring a variable and also a function anywhere in the code is equivalent to declaring it at the top. This means that a variable can appear to be used before it's declared as though the variable declaration was moved to the top of the function or global code.

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
In this instance `foo` is not declared as a variable so an Uncaught ReferenceError is thrown when `foo` is encountered. However the execution context knows about the `bar` declaration before the function is run and will initialise it to undefined.

Where variables are initialised to undefined at the top of the function scope only their names are hoisted. With function declarations the function body is hoisted as well. Consider the following code:

```js
function anotherHoistingTest() {
  foo(); // TypeError "foo is not a function"
  bar(); // this runs!
  var foo = function() { // this function expression is now assigned to local variable 'foo'
    console.log('this will not run');
  }
  function bar() { // function expression, given the name 'bar'
    console.log('this will run');
  }
}
anotherHoistingTest();
```
As of ES6+ variable hoisting has become less of a problem owing to the introduction of the `let` and `const` keywords. Attempting to use `let` or `const` values before they are declared will cause the interpreter to throw and error. This at least seems to solve the long-standing hoisting gotcha inherent in using the `var` keyword.
```js
function doSomething() {
  console.log(bar);
  const bar = 111; // using the let keyword gives the same result
}

doSomething() // Uncaught ReferenceError: bar is not defined
```

### Solution
To avoid unexpected behaviour with functions being available for use before they are declared, some developers prefer to declare all variables at the top of the enclosing scope. Furthermore declaring functions as variables is also often preferred to using named function expressions. for example
```js
// OK. Function body is hoisted meaning it can be used before declaration.
function thisNamedFunction() {};

// A bit better. `thisOtherNamedFunction` is initialised to undefined at the top of the enclosing scope but cannot be called until the function body assignment is encountered. This may result in more predictable behaviour.
var thisOtherNamedFunction = function() {};
```
Better still though is to use let or const keywords for variable declaration as this forces you to declare them before they are used avoiding any behind the scenes hoisting.

## The this keyword
Question: Explain this

Answer: At the time of execution of every function, a property called `this` is set and it refers to the current execution context. `this` always refers to an object and which object depends on the context in which the function is being called. For example:

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
* While executing a function in the context of an object, the object becomes the value of `this`
* Inside a setTimeout function, the value of `this` is the window object.
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
* If you use a constructor (by using the new keyword) to create an object, the value of `this` will refer to the newly created object.
* You can set the value of `this` to an object of your choosing by passing that object as the first argument of bind, call or apply
* For an event handler bound to a DOM element, value of `this` would be the element that the event was fired upon

## Constructing Primitive Types
```js
// Question: what do the following logs output?
const emptyConstructedString = new String('');
const emptyStringLiteral = '';
console.log(typeof emptyConstructedString);
console.log(typeof emptyStringLiteral);

// Answer: 
console.log(typeof emptyConstructedString); // "object"
console.log(typeof emptyStringLiteral); // "string"
```
### Discussion
Although we are passing empty string to the string constructor, calling `new String()` will naturally create a new object instance. The same is true when calling `new Boolean()`

## Equality checking
```js
// Question: Given the following variable assignments, What is the outcome of the following equality checks?

let foo;
const bar = null;

foo == bar;
foo === bar;

// Answer:
console.log(foo == bar); // true
console.log(foo === bar); // false
```
### Discussion
Double equals `==` tests for equality where triple equals `===` tests for equality and type.
The expression `let foo;` assigns the variable `foo` a value of `undefined` and so when comparing this value to `null` only by equality the JavaScript interpreter sees that both values are assigned to a nothing value and returns true.

However, comparing on equality and type returns `false` as `undefined` and `null` are different types in JavaScript
