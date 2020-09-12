---
title: 'Error Handling'
date: 2020-8-31 11:12:43
categories: 前端
tags: [The-modern-javascript-tutorial, js-language]
---

# `throw` operator
## standard errors

```javascript
// built-in constructors for standard errors
let error = new Error(message);
let error = new SyntaxError(message);
let error = new ReferenceError(message);
// ...
```

## syntax
<!--more-->
```javascript
let json = '{ "age": 30 }'; // incomplete data

try {
  let user = JSON.parse(json); // <-- no errors
  if (!user.name) {
    throw new SyntaxError("Incomplete data: no name"); // (*)
  }
} catch(e) {
  alert( "JSON Error: " + e.message ); // JSON Error: Incomplete data: no name
}
```

## Rethrowing
- `catch` should only process errors that it knows and “rethrow” all others (use `instanceof`)

```javascript
let json = '{ "age": 30 }'; // incomplete data

try {
  let user = JSON.parse(json);
  if (!user.name) {
    throw new SyntaxError("Incomplete data: no name");
  }
  blabla();           // unexpected error
} catch(e) {
  if (e instanceof SyntaxError) { 
    alert( "JSON Error: " + e.message );
  } else {
    throw e;          // rethrow (*)
  }
}
```

## `try...catch...finally`
- `finally` is executed even with an explicit `return`.
- `finally`  is executed just before returns to the outer code. 

```javascript
try {
   ... try to execute the code ...
} catch(e) {
   ... handle errors ...
} finally {
   ... execute always ...
}
```

### `try...finally`
-  Don’t want to handle errors.
-  But want to be sure that processes are finalized.

# custom errors, extending `Error`

```javascript
class ValidationError extends Error {
  constructor(message) {
    super(message);
    this.name = "ValidationError";
  }
}
```

## Wrapping exceptions
- a function handles low-level exceptions (`PropertyRequiredError("age")`),  and creates higher-level errors (` ReadError`) instead of various low-level ones.
