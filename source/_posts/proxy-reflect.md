---
title: 'Proxy & Reflect'
date: 2020-9-9 10:39:21
categories: 前端
tags: [The-modern-javascript-tutorial, js-language]
---

# Proxy
`let proxy = new Proxy(target, handler)`
1. `target`: is an object to wrap, can be anything, including functions.
2. `handler`: proxy configuration, an object with “traps”, methods that intercept operations. – e.g. get trap for reading a property of target, set trap for writing a property into target...

## empty handler
- As there are no traps, all operations on `proxy` are forwarded to `target`.
  1. writing operation `proxy.test=` sets the value on `target`
  2. reading operation `proxy.test` returns the value from `target`
  3. `for(key in proxy)` returns values from `target`
<!--more-->
## traps
<table>
	<thead><tr>
		<th>Internal Method</th>
		<th>Handler Method</th>
		<th>Triggers when…</th>
	</tr></thead>
	<tbody><tr>
		<td><code>[[Get]]</code></td>
		<td><code>get</code></td>
		<td>reading a property</td>
	</tr>
	<tr>
		<td><code>[[Set]]</code></td>
		<td><code>set</code></td>
		<td>writing to a property</td>
	</tr>
	<tr>
		<td><code>[[HasProperty]]</code></td>
		<td><code>has</code></td>
		<td><code>in</code> operator</td>
	</tr>
	<tr>
		<td><code>[[Delete]]</code></td>
		<td><code>deleteProperty</code></td>
		<td><code>delete</code> operator</td>
	</tr>
	<tr>
		<td><code>[[Call]]</code></td>
		<td><code>apply</code></td>
		<td>function call</td>
	</tr>
	<tr>
		<td><code>[[Construct]]</code></td>
		<td><code>construct</code></td>
		<td><code>new</code> operator</td>
	</tr>
	<tr>
		<td><code>[[GetPrototypeOf]]</code></td>
		<td><code>getPrototypeOf</code></td>
		<td><code>Object.getPrototypeOf</code></td>
	</tr>
	<tr>
		<td><code>[[SetPrototypeOf]]</code></td>
		<td><code>setPrototypeOf</code></td>
		<td><code>Object.setPrototypeOf</code></td>
	</tr>
	<tr>
		<td><code>[[IsExtensible]]</code></td>
		<td><code>isExtensible</code></td>
		<td><code>Object.isExtensible</code></td>
	</tr>
	<tr>
		<td><code>[[PreventExtensions]]</code></td>
		<td><code>preventExtensions</code></td>
		<td><code>Object.preventExtensions</code></td>
	</tr>
	<tr>
		<td><code>[[DefineOwnProperty]]</code></td>
		<td><code>defineProperty</code></td>
		<td><code>Object.defineProperty</code>, <code>Object.defineProperties</code></td>
	</tr>
	<tr>
		<td><code>[[GetOwnProperty]]</code></td>
		<td><code>getOwnPropertyDescriptor</code></td>
		<td><code>Object.getOwnPropertyDescriptor</code>, <code>for..in</code>, <code>Object.keys/values/entries</code></td>
	</tr>
	<tr>
		<td><code>[[OwnPropertyKeys]]</code></td>
		<td><code>ownKeys</code></td>
		<td><code>Object.getOwnPropertyNames</code>, <code>Object.getOwnPropertySymbols</code>, <code>for..in</code>, <code>Object/keys/values/entries</code></td>
	</tr></tbody>
</table>

## default value with `get` trap
To intercept reading, the `handler` should have a method `get(target, property, receiver)`.
1. `target`:  is the `target` object, the one passed as the first argument to `new Proxy(target, handler)`.
2. `item`: property name
3. `receiver`: `proxy` object itself.

- We can use `Proxy` to implement any logic for “default” values.
  1. non-existing array item returns `0` instead of `undefined`.
  2. non-existing key-value returns `defaultParse` instead of `undefined`. e.g.

```javascript
let dictionary = {
  'Hello': 'Hola',
  'Bye': 'Adiós'
};

dictionary = new Proxy(dictionary, {
  get(target, item) { // intercept reading a property from dictionary
    if (item in target) { // if we have it in the dictionary
      return target[item]; // return the translation
    } else {
      // otherwise, return the non-translated item
      return item;
    }
  }
});

alert( dictionary['Hello'] ); // Hola
alert( dictionary['Welcome to Proxy']); // Welcome to Proxy (no translation)
```

## Validation with “set” trap
To intercept setting sth, let's say we want an array exclusively for numbers. If a value of another type is added, there should be an error.
`set(target, property, value, receiver)`
1. `target`:  is the `target` object, the one passed as the first argument to `new Proxy(target, handler)`.
2. `property`: property name
3. `value`: property value
4. `receiver`: `proxy` object itself, matters only for setter properties.

- The set trap should return `true` if setting is successful, and `false` otherwise (triggers TypeError).

```js
let numbers = [];

numbers = new Proxy(numbers, { // (*)
  set(target, prop, val) { // to intercept property writing
    if (typeof val == 'number') {
      target[prop] = val;
      return true;
    } else {
      return false;
    }
  }
});

numbers.push(1); // added successfully
numbers.push("test"); // TypeError ('set' on proxy returned false)
```

> Don't forget to return `true`
>> There are `invariants` (conditions that must be fulfilled by internal methods and traps) to be held. 
>> e.g. `set` must return `true/false`
>> `delete` must return `true` if the value was deleted successfully...


> Built-in func of arrays ins still working
>> `proxy` doesn't break anything. 

## Iteration with `ownKeys` and `getOwnPropertyDescriptor`
### `ownKeys`
`Object.keys`, `for..in` loop and most other methods that iterate over object properties use `[[OwnPropertyKeys]]` (intercepted by `ownKeys` trap)
1. `Object.getOwnPropertyNames(obj)` returns non-symbol keys.
2. `Object.getOwnPropertySymbols(obj)` returns symbol keys.
3. `Object.keys/values()` returns non-symbol keys/values with `enumerable` flag.
4. `for..in` loops over non-symbol keys with `enumerable` flag + prototype keys.

- We can use `ownKeys` trap to make iteration, like `for...in` loop, `Object.keys`, `Object.values` work differently.
- Let's say skip properties starting with an underscore `_`

```js
let user = {
  name: "John",
  age: 30,
  _password: "***"
};

user = new Proxy(user, {
  ownKeys(target) {
    return Object.keys(target).filter(key => !key.startsWith('_'));
  }
});

// "ownKeys" filters out _password
for(let key in user) alert(key); // name, age
alert( Object.keys(user) ); // name,age
alert( Object.values(user) ); // John,30
```

### `getOwnPropertyDescriptor`
We can use the trap `getOwnPropertyDescriptor` to intercept calls to `[[GetOwnProperty]]`, and return a descriptor with `enumerable: true`.

```js
let user = { };

user = new Proxy(user, {
  ownKeys(target) { // called once to get a list of properties
    return ['a', 'b', 'c'];
  },

  getOwnPropertyDescriptor(target, prop) { // called for every property
    return {
      enumerable: true,
      configurable: true
      /* ...other flags, probable "value:..." */
    };
  }
});

alert( Object.keys(user) ); // a, b, c
```  

## Protected properties with `deleteProperty`
We can use proxies to prevent any access to properties starting with `_`, that conventionally means internal properties and methods.
1. `get` to throw an error when reading such property,
2. `set` to throw an error when writing,
3. `deleteProperty` to throw an error when deleting,
4. `ownKeys` to exclude properties starting with `_` from `for..in` and methods like `Object.keys`.

```js
let user = {
  name: "John",
  _password: "***"
};

user = new Proxy(user, {
  get(target, prop) {
    if (prop.startsWith('_')) {
      throw new Error("Access denied");
    }
    let value = target[prop];
    return (typeof value === 'function') ? value.bind(target) : value; // (*)
  },
  set(target, prop, val) { // to intercept property writing
    if (prop.startsWith('_')) {
      throw new Error("Access denied");
    } else {
      target[prop] = val;
      return true;
    }
  },
  deleteProperty(target, prop) { // to intercept property deletion
    if (prop.startsWith('_')) {
      throw new Error("Access denied");
    } else {
      delete target[prop];
      return true;
    }
  },
  ownKeys(target) { // to intercept property list
    return Object.keys(target).filter(key => !key.startsWith('_'));
  }
});
```

## `in range` with `has` trap
## Wrapping functions: `apply`
## `Reflect`

# Proxy limitations
# Revocable proxies