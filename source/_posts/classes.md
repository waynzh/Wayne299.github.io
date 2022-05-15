---
title: 'Classes'
date: 2020-8-30 12:20:10
categories: FE
tags: [js基础, The-modern-javascript-tutorial]
---

# Class basic
## syntax

```javascript
class MyClass { // basically is a function
  prop = value; // property, is not written to `MyClass.prototype`
  constructor(...) { ... } // constructor
  method(...) {} // method
  get something(...) {} // getter method
  set something(...) {} // setter method

  [Symbol.iterator]() {} // method with computed name (symbol here)
  ['say' + 'Hi']() {}    // `[...]` computed `sayHi()`
}
```
<!--more-->

# class field
## dynamic `this`
-  If an object method is passed around and called in another context, e.g. `setTimeout()`, this won’t be a reference to its object any more.
  1. we can pass a wrapper-function, such as `setTimeout(() => button.click(), 1000)`
  2. Bind the method to object. `click = () => {this...}` class field is created with `this` inside it referencing that object. We can pass `click` function around anywhere.

# Class inheritance
## `extends` keyword
- `class Rabbit extends Animal`: it sets `Rabbit.prototype.[[prototype]]` to `Animal.prototype` 

## `super` keyword
### overriding a method
- `super.method()` to call a parent method.
- `super()` to call a parent constructor (inside our constructor only)

```javascript
class Rabbit extends Animal {
	hide() {...}
	stop() {
	  super.stop(); // call parent stop
	  this.hide(); // and then hide
	}
}
```

### overriding constructor
- **Constructor in inheriting classes must call `super(...)`, and do it before using `this`.**

### internals, `[[HomeObject]]`
- When a function is specified as a class or object method, its `[[HomeObject]]` property becomes that object.
- it is defined for methods(not function properties) both in classes and in plain objects.
- for objects, methods must be specified exactly as `method()`, not as `method: function()`.

### overriding class fields
- When the parent constructor is called in the derived class, parent constructor always uses its own **field value**, not the overridden one.
- But it uses the overridden **method** instead.

# Static properties and methods
- Static properties are used when we’d like to store `class` level data,  like `User.property = ...` Usually create and clone methods.
- Not bound to the instance.
- `this == classItself`

```javascript
class User {
  static staticMethod() {
    alert(this === User);
  }
}
```

# Private and protected
## Protected, `_property`
- Protected properties are usually prefixed with an underscore `_`, that's a well-known convention, not enforced at the language level.

### Getter/setter functions
- Getter/setter functions are preferred, because flexibility and multiple arguments. E.g. `getNumber()`, `setNumber(num)`

## Private, `##property` 
- `##`: language level.
- Protected fields are naturally inheritable. Unlike private ones.
- We can’t access private field from outside or from inheriting classes.

# Extending built-in classes
## No static inheritance in built-ins
- Normally, when one class extends another, both static and non-static methods are inherited. 
- E.g. `Date` extends `Object`, so their instances have methods from `Object.prototype`.
- But `Date.[[Prototype]]` doesn't reference `Object`, so there is no `Date.keys()` static method. In other words, there is no link between `Date` and `Object`, so built-in classes don’t inherit statics from each other.
- That’s an important difference of inheritance between built-in objects compared to what we get with `extends`.

# Class checking: `instanceof`
## `instanceof` operator
### syntax
- `obj instanceof Class`: returns true/false
- constructor functions:`new Class() instanceof Class` 
- built-in classes: `arr instanceof Array` / `arr instanceof Object`
  1. `Array` prototypically inherits from `Object`
  2. `instanceof` examines the **prototype chain** for the check. 
  3. So when a prototype property is changed after the object is created `Animal.prototype={}`, `rabbit` is not a Animal anymore.

## other ways to check
- `typeof`: works for primitives, and return a string.
- `{}.toString`: works for primitives, built-in objects, objects with `Symbol.toStringTag`, and it returns a string. 

# Minxins
- a **mixin** is a class containing methods that can be used by other classes without a need to inherit from it.
- a **mixin** provides methods that implement a certain behavior, but we do not use it alone, we use it to add the behavior to other classes.
## syntax

```javascript
let sayMixin = {
  say(phrase) {
    alert(phrase);
  }
};

// mixin, can make use of inheritance inside themselves
let sayHiMixin = {
  __proto__: sayMixin, // or we could use Object.create to set the prototype here

  sayHi() {
    // call parent method
    super.say(`Hello ${this.name}`); // (*)
  },
  sayBye() {
    super.say(`Bye ${this.name}`); // (*)
  }
};

// usage:
class User {
  constructor(name) {
    this.name = name;
  }
}

// copy the methods
Object.assign(User.prototype, sayHiMixin);

// now User can say hi
new User("Dude").sayHi(); // Hello Dude!
```

- Even though `sayHi` and `sayBye` got copied, their `[[HomeObject]]` references `sayHiMixin` because they're created in it.
- So, `super` looks for parent methods in `[[HomeObject]].[[Prototype]]`
NOT `User.[[Prototype]]`.

## EventMixin 
