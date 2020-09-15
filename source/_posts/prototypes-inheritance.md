---
title: 'Prototypes / Inheritance'
date: 2020-9-3 12:11:20
categories: 前端
tags: [The-modern-javascript-tutorial, js-language]
---

# Prototypal inheritance
## `obj.__proto__`  -- old verison
- to access objects' hidden `[[Protptype]]` property, either another object or `null`.

## Writing doesn't use prototype
- Write / delete operator work directly with the object. Not use to change the prototype.
- The prototype is only used for reading properties

## iterate properties
- `for...in` loop iterates over inherited properties.
- `Object.keys()` only returns own keys. (almost all key/value-getting methods)
<!--more-->

## `F.prototype`, constructor property
### `F.prototype`
- Every function has the "prototype" property even if we don’t supply it.
- On regular objects the `prototype` is nothing special.

```javascript
let animal = {           // Object
  eats: true
};

function Rabbit(name) {  // Function
  this.name = name;
}

Rabbit.prototype = animal;

let rabbit = new Rabbit("White Rabbit"); //  rabbit.__proto__ == animal
```

### constructor property
- The default `prototype` of function is an `object` with the only property `constructor` that points back to the function itself.
- `F.prototype.constructor == F`

```javascript
function Rabbit() {}

Rabbit.prototype = {   // default
 	constructor: Rabbit
};

Rabbit.prototype.jumps = true 
// Not overwrite F.prototype totally, keep the defualt Rabbit.prototype.constructor 
```

- We can use `let rabbit2 = new rabbit.constructor("Black Rabbit")` when we don’t know which constructor was used for it.

## `null` and `undefined` have no object wrappers
-  There are no corresponding prototypes either.

## Example ??

```javascript
Function.prototype.defer = function(ms) {
  // `this` is passed to the original method
  let f = this;      

  return function(...args) {
    // `this` to make decoration work for object methods
    setTimeout(() => f.apply(this, args), ms);  

  }
};

let user = {
  name: "John",
  sayHi() {
    alert(this.name);
  }
}

user.sayHi = user.sayHi.defer(1000);

user.sayHi();
```

# Property methods
- `Object.create(proto, descriptors)`: creates an empty object with given `proto` as `[[Prototype]]` and optional property descriptors. (descriptors are in the same format as Object property flags and descriptors).

```javascript
let animal = {
  eats: true
};

let rabbit = Object.create(animal, {
  jumps: {
    value: true,
    // enumberable: true,
    // writable: true,
    // configuarable: true,
  }
});

alert(rabbit.jumps); // true
```

- `Object.getPrototypeOf(obj)`: returns the `[[Prototype]]` of obj.
- `Object.setPrototypeOf(obj, proto)`: sets the `[[Prototype]]` of obj to `proto`.

## "Very plain" objects
- `let obj = Object.create(null)`: with no build-in methods and without a prototype.
- have no issues with `__proto__` as the key.


# Example

```javascript
function Rabbit(name) {
  this.name = name;
}
Rabbit.prototype.sayHi = function() {
  alert(this.name);
};

let rabbit = new Rabbit("Wayne");
rabbit.sayHi();            // Wayne (this = rabbit)
Rabbit.sayHi();            // undefined (this = Rabbit.prototype)
Rabbit.prototype.sayHi();  // undefined
rabbit.__proto__.sayHi();  // undefined
```
- 