---
title: 'Functions'
date: 2020-9-1 21:20:12
categories: 前端
tags: [The-modern-javascript-tutorial, js-language]
---

# Rest parameters & spread syntax
## Rest parameters, `...`

```js
function sumAll(...args) { // args is the name for the array
  let sum = 0;
  for (let arg of args) {
  	sum += arg;
  }
  return sum;
}

alert( sumAll(1) ); // 1
alert( sumAll(1, 2) ); // 3
alert( sumAll(1, 2, 3) ); // 6
```

## `arguments` variable --old version
<!--more-->
- it's iterable and array-like.
- Arrow functions do NOT have `arguments`, it takes the outer function.

## Spread syntax `...arr`
- When `...arr` is used in the function call, it “expands” an iterable object `arr` into the list of arguments.
- Any iterable will do the same.



```javascript
let arr1 = [1, -2, 3, 4];
let arr2 = [8, 3, -8, 1];

// spread turns array into a list of arguments
// Math.max(arr[0], arr[1], arr[2]...)
// can pass multiple iterables
Math.max(...arr1, ...arr2)   // 8
```

- Can combine the spread syntax with normal values:
  `let merged = [0, ...arr, 2, ...arr2];`
  
- `Array.from`: operates on both array-likes and iterables. -- more universal
- `[...obj]`: spread syntax works only with iterables.
  
- Get a new copy: much shorter than `objCopy = Object.assign({}, obj)`

```javascript
let arr = [1, 2, 3];
let arrCopy = [...arr]; // different reference

let obj = { a: 1, b: 2, c: 3 };
let objCopy = { ...obj };  // different reference
```

# Closure
- Defination:  a function that remembers its outer variables and can access them.
- in JavaScript, all functions are naturally closures (there is only one exception, to be covered in The "new Function" syntax).
-  they automatically remember where they were created using a hidden `[[Environment]]` property, and then their code can access outer variables.

## Confusing issue

```javascript
let phrase = "Hello";

if (true) {
  let user = "John";

  function sayHi() {
    alert(`${phrase}, ${user}`);
  }
}

sayHi(); // error! It only lives inside the if block

```

```javascript
let x = 1;

function func() {
  console.log(x); // ReferenceError: Cannot access 'x' before initialization `let`
  let x = 2;
}

func();
```

```javascript
function makeCounter() {
  let count = 0;

  return function() {
    return count++;
  };
}

let counter = makeCounter();
let counter2 = makeCounter();

alert( counter() ); // 0
alert( counter() ); // 1

/** 
`counter` and `counter2` are created by different invocations of `makeCounter`. (not the same with Object, which based on the reference)
So they have independent outer Lexical Environments, each one has its own count.
*/

alert( counter2() ); // 0
alert( counter2() ); // 1
```

```javascript
let arr = [1, 2, 3, 4, 5, 6, 7];

// return a function to return true
function inBetween(a, b) {
    return function (item) {
        return item <= b && item >= a;
    };
}

function inArray(arr) {
    return function (item) {
        return arr.includes(item);
    };
}

console.log(arr.filter(inBetween(3, 6))); // 3,4,5,6
console.log(arr.filter(inArray([1, 2, 10]))); // 1,2
```

# var 
## `var` tolerates redeclarations
## `var` hoisting
- it can be declared below their use.
- But assignments are not.

## IIFE (immediately-invoked function expressions)
- `(function() {...}) ()` 
- `(function() {...} ())`
- `!function() {...} ()`
- `+function() {...} ()`

## Named Function Expression ??
## new Function
### syntax
- `let func = new Function ([arg1, arg2, ...argN], functionBody);`

```javascript
// the function is created literally from a string.
let sum = new Function('a', 'b', 'return a + b');

alert( sum(1, 2) ); // 3
```

- We can receive a new function from a server and then execute it.

### closure
- `[[Environment]]` referencing the global Lexical Environment, not the outer one.
- Doesn’t have access to outer variables, only to the global ones.

# Decorator
## Transparent caching

```javascript
function slow(x) {
  // there can be a heavy CPU-intensive job here
}

function cachingDecorator(func) {
  let cache = new Map();

  return function(x) {     // return the function
    if (cache.has(x)) {    // if there's such key in cache
      return cache.get(x); // read the result from it
    }

    let result = func(x);  // otherwise call original func

    cache.set(x, result);  // and cache (remember) the result
    return result;
  };
}

slow = cachingDecorator(slow);  // use cache
```

- But not suitable for `Object`, because without the context `this`
- `Object` need to use `func.call()` method.

####for Object `func.call(this, ...args)`

```javascript
function say(phrase) {
  alert(this.name + ': ' + phrase);
}

let user = { name: "John" };

// user becomes `this`, and "Hello" becomes the first argument
say.call( user, "Hello" ); // John: Hello
```

- for object, we can modify the above example by changing to
`let result = func.call(this, x); // "this" is passed correctly`

## call forwarding
### syntax `func.apply(this, args)`
- `args`: array-like object.
- For objects that are both iterable and array-like, like a real array, we can use any of them, but apply will probably be faster, because most JavaScript engines internally optimize it better.

```javascript
let wrapper = function() {
  return original.apply(this, arguments);
};
```

## method borrowing
- In this example. we borrow a join method from a regular array `[].join` and use this func to `func.call` the context of `arguments`

```javascript
function hash() {
  alert( [].join.call(arguments) ); // 1,2
}

hash(1, 2);
```

- Usually used in taking **array methods** and apply them to `arguments`

## Example ??
- Debounce decorator: runs the function once after the “cooldown” period. Good for processing the final result. e.g. show the result of typing.

```javascript
function debounce(func, ms) {
  let timeout;
  return function() {
    clearTimeout(timeout);  // cancel previous such timeout.
    timeout = setTimeout(() => func.apply(this, arguments), ms); // After the last call, then runs its function
  };
}
```

- Throttle decorator: runs it not more often than given time. Good for regular updates that shouldn’t be very often. e.g. trace mouse movement. ??

# Function binding
## Solution 1: a wrapper

```javascript
let user = {
  firstName: "John",
  sayHi() {
    alert(`Hello, ${this.firstName}!`);
  }
};

setTimeout(function() {
  user.sayHi(); // Hello, John!
}, 1000);
```

- but if the value changes within the timeout, it'll call the wring object.

## solution 2: bind
### Syntax `let boundFunc = func.bind(context);`
- `boundFunc`: return a function-like “exotic object”, that is callable as function but doesn't have other properties.
- `context`: set `this=context`.
- A function cannot be re-bound.

```javascript
let user = {
  firstName: "John",
  sayHi() {
    alert(`Hello, ${this.firstName}!`);
  }
};

let sayHi = user.sayHi.bind(user); // (*)

// can run it without an object
sayHi(); // Hello, John!

setTimeout(sayHi, 1000); // Hello, John!

// even if the value of user changes within 1 second
// sayHi uses the pre-bound value which is reference to the old user object
user = {
  sayHi() { alert("Another user in setTimeout!"); }
};  // not affected by this statement
```

### Partial functions `func.bind(context, ...args)`
- When we have a very generic function and want a less universal variant of it for convenience.
- e.g. `send(from, to, text)` => `sendTo(to, text)` using partial function.

# Arrow functions
## Don't have `this`
- Convinent to iterate inside an object method.

## Can't be used as `constructor`
- Can't be called with `new`

## No `arguments` variable
## Don't have `super`




