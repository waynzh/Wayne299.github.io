---
title: 'JavaScript Miscellaneous'
date: 2020-9-9 10:39:21
categories: FE
tags: [js基础, The-modern-javascript-tutorial]
---

# Currying
Translates a function from callable as `f(a, b, c)` into callable as `f(a)(b)(c)`.

## Example

```js
log(new Date(), "DEBUG", "some debug"); // log(a, b, c)

// can make a convenience function
let logNow = log(new Date()); // [HH:mm]

logNow("INFO", "message"); // [HH:mm] INFO message 
```

-  `logNow` is `log` with fixed first argument, in other words “partially applied function” or “partial” for short.
<!--more-->
## Advanced curry implementation ??

# Reference Type
The value of Reference Type is a three-value combination (base, name, strict), where:
1. `base` is the object.
2. `name` is the property name.
3. `strict` is `true` if `use strict` is in effect.

- The result of a property access `user.hi` is not a function, but a value of **Reference Type**. 

```js
strict mode
// Reference Type value of `user.hi`
(user, "hi", true)

hi = user.hi 
// discards the reference type as a whole, takes the value of this function and passes it on.
// So any further operation “loses” `this`
```

- **Reference type** is a special “intermediary” internal type, with the purpose to pass information from dot `.` to calling parentheses `()`.

# BigInt
BigInt is created by appending `n` to the end of an integer literal or by calling the function `BigInt()` that creates bigints from strings, numbers etc.

- All operations on bigints return bigints. `5n / 2n  // 2`
- We can’t mix bigints and regular numbers.
- The unary plus `+value` is not supported on bigints, we should use `Number()` to convert a bigint to a number.




# Eval: run a code string
## syntax
- `eval` is the result of the last statement.
- `eval` is executed in the current lexical environment, so it can see outer variables -- That’s considered bad practice.
- In strict mode, `eval` has its own lexical environment. So functions and variables, declared inside `eval`, are not visible outside

```js
let value = eval('let i = 0; ++i');
alert(value); // 1
```

## Using `eval`
- Now, we can replace it with a modern language construct or a JavaScript `Module`.
- To `eval` the code in the global scope, use `window.eval(code)`.
- Needs some data from the outer scope, use `new Function('a', 'alert(a)')` and pass it as arguments.



