# Prototypal Inheritance
## Notes on inheritance and the prototype chain
The ES6 `class` keyword will make working with inheritance more familiar to developers coming from a class-based or object oriented language background. JavaScript, however, is prototype-based and the class keyword is merely a familiar interface or _syntactic sugar_ for JavaScript's prototypal inheritance system. Inheritance can therefore be acheived without classes and class inheritance.

Each object in JavaScript is created with a link to it's own prototype object. That prototype object can in turn have its own prototype object and so on until the root is reached with a `null` value set as its prototype. `null` is an object type in JavaScript but has no prototype object by design.

When a portion of your program tries to access properties or methods on a given object, the property will be looked for first on the object itself and then on the object's prototype and then the prototype's prototype and so on until the accessor has either found the property being searched for or reached the `null` object at the root of the prototype chain. Should null be reached the accessor will return `undefined`.

## Table of contents
- [The prototype chain and literal syntax construction](#the-prototype-chain-and-literal-syntax-construction)
- [The prototype chain and the `new` keyword](#the-prototype-chain-and-the-new-keyword)

## The prototype chain and literal syntax construction
```js
var obj = {a: 1};
// obj ---> Object.prototype ---> null

var arr = ['foo', 'bar', 'baz'];
// inherits first from Array.prototype (indexOf, forEach, map, etc)
// arr ---> Array.prototype ---> Object.prototype ---> null

function f(a, b) {
  return a + b;
}

// inherits first from from Function.prototype (call, bind, etc)
// f ---> Function.prototype ---> Object.prototype ---> null
```

## The prototype chain and the `new` keyword

```js
function List() {
  this.items = [];
}

List.prototype = {
  addItem: function(item) {
    this.items.push(item);
  }
};

var shopping = new List();
// List is just a function. Constructors in JavaScript are just plain functions with the first letter capitalised by developers as a convention.
// shopping is an object with own property items.
// the shopping prototype is the value of List.prototype when new List() is called.
// shopping ---> List.prototype --> Function.prototype ---> Object.prototype ---> null
```