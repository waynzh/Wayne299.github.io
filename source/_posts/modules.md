---
title: 'Modules'
date: 2020-9-9 00:20:10
categories: 前端
tags: [The-modern-javascript-tutorial, js-language]
---

# Basic
## syntax
- In index.html, we must tell the browser that a script should be treated as a module, by using the attribute `<script type="module">`
- Always `use strict` and Each module has its own module-level scope. Only `export` what they want to be accessible from outside and `import` what they need.
- Modules work only via HTTP(s), not in local files.

## A module code is evaluated only the first time when imported
- All importers get exactly the one and only that export, so it's shared between importers

## import.meta
- `import.meta`: contains the information about the current module.
<!--more-->

## `this` is undefined
## Module `<script>`s are deferred
-  Wait until the HTML document is fully ready.

## `async` works on inline scripts
`<script async type="module">`: inline script has `async` doesn’t wait for anything.
- used for counters, ads, document-level event listeners...

# Export and Import
- `import/export` statements don’t work if inside `{...}`. As for importing conditionally, look at the **Dynamic Imports** part.

## import
### import *
- We can import everything as an object using `import * as <obj> from src`.

### import "as"
`import {sayHi as hi, sayBye as bye} from './say.js'` for brevity.

## Export
- No semicolons after `export class/function` 
- Technically we could put `export` above functions declarations as well. `export {sayHi, sayBye}; // a list of exported variables`

### export "as"
`export {sayHi as hi, sayBye as bye}`

### export default
`export default class User {}` then we can import without curly braces, `import User form './user.js'`
- We can choose the name `User` or `myUser` when importing. But there is the convention that imported variables should correspond to file names.
- At most one `default export` per file, so no name is fine because we don't need to `{name}`. Import without curly braces knows what to import.

## The “default” name
- `export {sayHi as default}` is the same as `export default sayHi`
- `import {default as User, sayHi} from './user.js'`: import the default export along with a named one
- `import * as user from './user.js'`: if import everthing `*`, then the `default` property is exactly the default export `let User = user.default`

## Re-export
`export...from...`: allows to import things and immediately export them (possibly under another name)
-  only named exports (not )

```javascript
// import login/logout and immediately export them
import {login, logout} from './helpers.js';
export {login, logout};

// shorter notation for such import-export
export {login, logout} from './helpers.js';
```

### Re-exporting the default export
We have to write `export {default} from ...`, for no named default.
- If we’d like to re-export both named and the default export, then two statements are needed:
  1. `export * from ...`
  2. `export {default} from ...`
  

# Dynamic Imports
- Dynamic imports work in regular scripts.

## The `import(module)` expression
It can be like `let {hi, bye} = await import('./say.js')` or `let {default: say} = await import('./say.js')` if inside a async function.
- It's not a function call, it’s a special syntax that just happens to use parentheses.
