---
title: 'Generators & Advanced iteration'
date: 2020-9-8 12:53:41
categories: FE
tags: [js基础, The-modern-javascript-tutorial]
---

# Generators
- Can return (“yield”) multiple values, one after another, on-demand. 
- They work great with `iterables`, allowing to create data streams with ease.

## generator functions, `function*`

```javascript
function* generateSequence() {
  yield 1;
  yield 2;
  return 3;
}
```

- When it (`generateSequence()`) is called, it doesn’t run its code. Instead it returns a “generator object” (`[object Generator]`), to manage the execution.
<!--more-->

### `next()`
- The main method of a generator is `next()`. When called, it runs the execution until the nearest `yield <value>` statement (`<value>` can be omitted), then the yielded `<value>` is returned to the outer code.
- The result of `next()` is an object with two properties: 
  1. `value`: the yielded value.
  2. `done`: `true` if the function code has finished (reache the `return` statement), otherwise `false`.

## generator are `iterable`
### `for...of`
- We can loop over using `for...of`
- but it IGNOREs the last value, when `done: true`. So we must return them with `yield` not `return`.

### `...` spread syntax
`let sequence = [0, ...generateSequence()] // 0, 1, 2, 3`
- It turns the iterable generator object into an array of items.
- we can call all related functionality, as generators are iterable.

## Using generators for iterables

```javascript
let range = {
  from: 1,
  to: 5,

  // a shorthand for [Symbol.iterator]: function*() which returns a generator
  *[Symbol.iterator]() { 
    for(let value = this.from; value <= this.to; value++) {
      yield value;
    }
  }
};

alert( [...range] ); // 1,2,3,4,5
```

## Generator composition, `yield*`
- “embed” (compose) one generator into another.

## `yield` is a two-way street, `generator.next(arg)`
- it also can pass the value inside the generator. And `arg` becomes the result of yield.

```javascript
function* gen() {
  // Pass a question to the outer code and WAIT for an answer
  let result = yield "2 + 2 = ?";  // (*)

  alert(result);  // onces (**) called, then stop wait and return value
}

let generator = gen();
let question = generator.next().value; // "2 + 2 = ?" <-- yield returns the value

generator.next(4); // (**) result = 4 --> pass the result into the generator
```

1. The first call `generator.next()` should be always made without an argument, It starts the execution and returns the result of the first `yield` `"2+2=?"`.
2. Pauses the execution, while staying on the line (*)
3. On `generator.next(4)`, the generator resumes, and `4` gets in becomes the result of yield: `result = 4`.

- It’s like a “ping-pong” game. Each `next(value)` (excluding the first one) passes a value into the generator, that becomes the result of the current `yield`
- then gets back the result of the next yield.

## pass an error into `yield`, `generator.throw`

```javascript
let generator = gen();
let question = generator.next().value;

generator.throw(new Error("Error!!")); 
```

# Async iterators and generators
## Async generators

```javascript
async function* generateSequence(start, end) {
  for (let i = start; i <= end; i++) {
    // values comes with a delay of 1 second between them
    await new Promise(resolve => setTimeout(resolve, 1000));

    yield i;
  }
}

(async () => {
  let generator = generateSequence(1, 5);
  let result = await generator.next(); // add await
  // or
  for await (let value of generator) {
    alert(value); // 1, then 2, then 3, then 4, then 5
  }
})();
```

## Async iterables
To make the “regular” iterable object iterable asynchronously:
- we need to use `Symbol.asyncIterator` and `for await (let item of iterable)` and turn a `promise` 
- and spread snytax `...` doesn't work saynchronously.

## Real-life example
- paginated requests: async generator that returns values.
- when we meet streams of data, we can use **async generators** to process such data. In browsers, there’s also another API called Streams.
