---
title: 'Promises & async/await'
date: 2020-9-7 19:20:43
categories: 前端
tags: [The-modern-javascript-tutorial, js-language]
---

#  Promise
##  Syntax

```javascript
let promise = new Promise(function(resolve, reject) {
  // executor (the producing code, `resolve()` / `reject()`)
});
```

- callbacks: There can be only one result or one error
  1. `resolve(value)`: if success, with result `value`
  2. `reject(error)`: if error, `error` is the error object.
<!--more-->

- internal properties:
  1. `state`: pending and settled
    * `pending`: initially
    * `fulfilled`: when `resolve` is called.
    * `rejected`: when `reject` is called.
  2. `result`:
    * `undefined`: initially
    * `value`: `resolve(value)` called.
    * `error`: `reject(error)` called.

##  handlers: `then`, `catch`, `finally`
- If a promise is returned, then JavaScript waits for it to settle and calls next handlers with its result.

###  `.then(result => {}, error => {})`
###  `.catch(f)`
- the same as `.then(null, f)`

###  `.finally(f)`
- the same as `.then(f,f)`
- good handler for perfprming cleanup.

##  Promises chaining
- The `value` (`return value` or `resolve(value)`) that first `.then` returns is passed to the next `.then` handler.
- Because `promise.then` returns a promise, so we can call the next `.then` on it.

###  Situation
* returns a value, `newPromise` gets resolved with the `returned value` as its value;
* doesn't return anything, `newPromise` gets resolved with an `undefined` value;
* throws an error, `newPromise` gets rejected with the `thrown error` as its value;


##  Returing promises
- `return new Promise()`: further handlers wait until it settles, and then get its result and pass along.

```javascript
new Promise(function (resolve, reject) {
    setTimeout(() => resolve(1), 1000);
}).then(function (result) {
    console.log(result); // 1; after 1s

    return new Promise((resolve, reject) => {
        setTimeout(() => resolve(result * 2), 1000);
    });
}).then(console.log); //  (*) 2; after 2s

```

###  .then(f)
- When passing a function `console.log()` as a parameter to some other function `then()`, the `argument list` of `console.log()` is always ignored
- because JavaScript picks up the arg-list of passed function automatically. 
- (*) e.g. `then(console.log)`: pass the parameters of `then` to `console.log()`, so ignore the parentheses.

#  Error handling with promises
- When a promise rejects, the control jumps to the closest rejection handler.
- So append `.catch` to the end of chain to catch all errors.

##  `.then` versus `.catch`
`promise.then(f1).catch(f2)` vs. `promise.then(f1, f2)` Are these code fragments equal? 

- no, they are not equal:
  1. If the promise itself gets rejected, then yes the functions will behave the same and the `f2` handlers will handle the error the same.
  2. But if an error occurs within the `f1` function itself after the promise has resolved, then only the `f2` function inside the catch will handle that error.
- The current `then()` handles the PREVIOUS promise in the chain. 
- When a promise goes error, throwing in `f1` can only be handled by the NEXT chained handler. 

## rethrowing

```javascript
// the execution: catch -> catch
new Promise((resolve, reject) => {
  throw new Error("Whoops!");
}).catch(function(error) { 
  if (error instanceof URIError) {
    // handle it
  } else {
    alert("Can't handle such error");
    throw error; // throwing this error jumps to the next catch
  }
}).then(function() {
  /* doesn't run here */
}).catch(error => { 
  alert(`The unknown error has occurred: ${error}`);
  // don't return anything => execution goes the normal way
});
```

#  Promise API
##  Promise.all
- e.g. download several URLs in parallel and process the content once they are all done.

###  syntax
`let promise = Promise.all([...promises...])` 
- not just `[]` any iterable is fine. If any of those objects is not a `promise`, it’s passed to the resulting array “as is” (not change).

- the order of the resulting array members is the same as in its source promises.
- If any of the promises is rejected, the promise returned by `Promise.all` immediately rejects with that error. But the others will still continue to execute, but `Promise.all` won’t watch them anymore, and their results will be ignored.
- 


- Trick: map an array of rough data into an array of promises, and wrap that into `Promise.all()`
  * e.g. `url = [...]; request = url.map(url => fetch(url)); Promise.all(request)`

##  Promise.allSettled
- `Promise.all` is good for “all or nothing” cases, but for `Promise.allSettled` just waits for all promises to settle, regardless of the result.

###  result array
- `{status:"fulfilled", value:result}` for successful responses
- `{status:"rejected", reason:error}` for errors.

##  Promise.race
- Waits only for the **first(fastest) settled promise** and gets its result (or error).
- all further results/errors are ignored.

### syntax
`let promise = Promise.race(iterable)`

## Promise.resolve/reject -- old version
- it is used when a function is expected to return a `promise`.
- e.g. thenables and functions we want write `loadCached(url).then(...)`, and guarantee to return a `promise`.


# Promisification
- `promisify(f, manyArgs = false)`

# Microtasks
## Microtasks queue
- Promise handlers `.then` `.catch` `.finally` are always asynchronous.
- So even when a `Promise` is immediately resolved, the code on the lines below `.then` `.catch` `.finally` will still execute before these handlers.

That's because:
- Execution of a task in this queue is initiated only when nothing else is running.
- So when a promise is ready, its handlers are put into the queue; they are not executed yet. When the JavaScript engine becomes free from the current code, it takes a task from the queue and executes it.

# Async/await
## Async functions, `async`
- `async` before a function means a function always returns a `promise`.

## Await, `await`
- works ONLY inside `async` functions: `let value = await promise`
- wait until that promise settles and returns its result.

```javascript
async function f() {
  let promise = new Promise((resolve, reject) => {
    setTimeout(() => resolve("done!"), 1000)
  });

  let result = await promise; // wait until the promise resolves (*)
  alert(result); // "done!"
}

f();
```

- It’s just a more elegant syntax of getting the promise result than promise.then, easier to read and write.

### error handling
- use a regular `try...catch`.
- But at the top level of the code, when we’re outside any `async` function, we’re syntactically unable to use `await`, so it’s a normal practice to add `.then/catch`
- works well with `Promise.all`: `let results = await Promise.all([...])`

### examples

```javascript
async function showAvatar() {

  // read our JSON
  let response = await fetch('/article/promise-chaining/user.json');
  let user = await response.json();

  // read github user
  let githubResponse = await fetch(`https://api.github.com/users/${user.name}`);
  let githubUser = await githubResponse.json();

  // show the avatar
  let img = document.createElement('img');
  img.src = githubUser.avatar_url;
  img.className = "promise-avatar-example";
  document.body.append(img);

  // wait 3 seconds
  await new Promise((resolve, reject) => setTimeout(img.remove(), 3000));

  return githubUser;
}

showAvatar();
```

### `await` accepts "thenables"

```javascript
class Thenable {
  constructor(num) {
    this.num = num;
  }
  then(resolve, reject) {
    alert(resolve);
    // resolve with this.num*2 after 1000ms
    setTimeout(() => resolve(this.num * 2), 1000); // (*)
  }
};

async function f() {
  // waits for 1 second, then result becomes 2
  let result = await new Thenable(1);
  alert(result);
}

f();
```























