# Prototypal Inheritance

## Table of contents
- [The prototype chain and literal syntax construction](#the-prototype-chain-and-literal-syntax-construction)
- [The prototype chain and the `new` keyword](#the-prototype-chain-and-the-new-keyword)
- [The prototype chain and `Object.create`](#the-prototype-chain-and-object.create)
- Inheritance Patterns
  - [Delegation](#delegation)
  - [Concatenative Inheritance](#contatenative-inheritance)
  - [Functional Inheritance](#functional-inheritance)
  - [Credits](#credits)

## Notes on inheritance and the prototype chain
The ES6 `class` keyword will make working with inheritance more familiar to developers coming from a class-based or object oriented language background. JavaScript, however, is prototype-based and the class keyword is merely a familiar interface or _syntactic sugar_ for JavaScript's prototypal inheritance system. Inheritance can therefore be acheived without classes and class inheritance.

Each object in JavaScript is created with a link to it's own prototype object. That prototype object can in turn have its own prototype object and so on until the root is reached which has a `null` value set as its prototype. `null` is an object type in JavaScript but has no prototype object by design.

When a portion of your program tries to access properties or methods on a given object, the property will be looked for first on the object itself and then on the object's prototype and then the prototype's prototype and so on until the accessor has either found the property being searched for or reached the `null` object at the root of the prototype chain. Should null be reached the accessor will return `undefined`.

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
// the shopping prototype is the value of List.prototype when new List() is called. Note that the ES6 `class` keyword is merely syntactic sugar over the above example.
// shopping ---> List.prototype --> Function.prototype ---> Object.prototype ---> null
```

## The prototype chain and `Object.create`

```js
const obj1 = {a: 1}; 
// obj1 ---> Object.prototype ---> null

const obj2 = Object.create(obj1);
// obj2 ---> obj1 ---> Object.prototype ---> null
console.log(obj2.a); // 1 (inherited)

const obj3 = Object.create(obj2);
// obj3 ---> obj2 ---> obj1 ---> Object.prototype ---> null
```

## Inheritance Patterns

### Delegation

The ES6 class keyword will provide a familiar interface to class construction for developers from class-based languages. But this approach can create brittle hierachies and neglects JavaScripts powerful prototype system.

```js

class Person {
  constructor(name) {
    this.name = name;
  }

  sayHello() {
    return `Hello, my name is ${this.name}`;
  }
}

const billy = new Person('Billy');
billy.sayHello() // Hello, my name is Billy
```
Delegation can also be achieved with `Object.create`. The pattern below creates a function `createPerson` that returns an object. This is known as a _factory_ function.
```js

const proto = {
  sayHello () {
    return `Hello, my name is ${ this.name }`;
  }
};

const createPerson = (name) => Object.assign(Object.create(proto), {
  name
});

const billy = createPerson('billy');

billy.sayHello(); // Hello, my name is Billy

```
One drawback to the above approach is that any changes made to the `proto` object declared above will be delegated to all delegate objects created by `createPerson`.


### Concatenative Inheritance / Mixins
Concatenative inheritance copies properties from one object to another but in doing so creates a clone of the target object. This means that any connection between the objects is lost and a fresh copy of the object is created each time you inherit. Although similar to our factory function above note how instead of using `Object.create` to inherit from `proto` we instead copy `proto` and our desired properties to a new object using `Object.assign`.
```js
const proto = {
  sayHello: function sayHello() {
    return `Hello, my name is ${ this.name }`;
  }
};

const billy = Object.assign({}, proto, { name: 'Billy' });

billy.sayHello(); // Hello, my name is Billy
```
The above approach proposes a way of blending different objects together or _mixing in_ properties of other objects to compose new object combinations. This is why this pattern is known as a _mixin_. Futhermore this pattern offers increased flexibility for the developer. When you consider that concatenative inheritance yields a 'has a' relationship with other objects as opposed to an 'is a' relationship the prototype offered by delgative inheritance pattern, the developer is afforded great latitude to combine the functionality of many objects types free from the constraints of strict class hierarchies imposed by the `class` keyword. For example, if we take our object above and we wanted to mix in the properties of a third object type we could do this with a mixin pattern.
```js
function Person() {};
Person.prototype.sayHello = function() {
  console.log(`Hello, my name is ${ this.name }`);
};

const mixin = Object.assign({
  sayAge() {
    console.log(`Hello, my age is ${ this.age }`);
  }
}, Person.prototype);

const billy = { name: 'Billy', age: 50 };
const model = Object.assign(billy, mixin);
billy.sayHello(); // Logs Hello my name is Billy
billy.sayAge() // Logs Hello my age is 50
```
The above example although a little crude demonstrates the inherent flexibility of the mixin pattern and how it proposes a compositional approach to shared functionality.

### Functional Inheritance
Similar to the concatenative inheritance above, functional inheritance offers the added flexibility of a closure to encapsulate any private state we need. When combined with `call` we can build up a flexible and expressive factory for object creation.
```js

function Person() {};
Person.prototype.sayHello = function() {
  console.log(`Hello, my name is ${ this.name }`);
};

const createPerson = function() {
  const attributes = { age: 'unknown' };
  return Object.assign(this, {
    setValue(name, value) {
      attributes[name] = value;
    },
    getValue(name) {
      return attributes[name];
    }
  }, Person.prototype)
};

const billy = { name: 'Billy' };
const personModel = target => createPerson.call(target);
const billyModel = personModel(billy);

billyModel.sayHello();
```

Now let us assume we need to hold some private internal state in our person objects. Functional inheritance gives us a closure that allows us to hide away private data and then access it only via priviledged methods. Let's imagine we need to avoid explicitly declaring the person's age at instantiation time. Following from the above example we can do

```js
const janet = { name: 'Janet' };
const janetModel  = personModel(janet);
janetModel.setValue('age', 22);
console.log(janetModel.getValue('age'));
```

## Credits
* [Inheritance and the prototype chain on developer.mozilla.org](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Inheritance_and_the_prototype_chain)
* [3 different kinds of prototypal inheritance: ES6 Edition on JavaScript Scene](https://medium.com/javascript-scene/3-different-kinds-of-prototypal-inheritance-es6-edition-32d777fa16c9)