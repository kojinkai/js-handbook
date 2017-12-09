# JS Gotchas

## Table of contents

- [Closures and SetTimout](#closures--settimeout)
- [Undefined vs NULL](#undefined-vs-null)
- [Variable Hoisting](#variable-hoisting)

## Closures & SetTimeout

```js
// Q: What will the following code output?
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
Variable declarations (and declarations in general) are processed before any code is executed and so declaring a variable anywhere in the code is equivalent to declaring it at the top.
This means that a variable can appear to be used before it's declared. This behavior is called "hoisting", as it appears that the variable declaration is moved to the top of the function or global code.

The above `doSomething()` function is implicitly understood as
```js
function doSomething() {
  var bar;
  console.log(bar); // undefined
  bar = 111;
  console.log(bar); // 111
}
```

Consider the following example with undeclared variables:
```js
function doSomethingElse() {
  console.log(foo);
  console.log(bar);
  var bar = 111;
}

doSomethingElse() // Uncaught ReferenceError: foo is not defined
```
So we can see that the execution context knows about any declarations before a function is ran and will initialise variables to undefined.