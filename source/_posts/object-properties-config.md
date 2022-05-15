---
title: 'Object Properties Configuration'
date: 2020-9-2 10:34:41
categories: FE
tags: [js基础, The-modern-javascript-tutorial]
---

# Accessor Property 
## getters and setters
- `get`:  a function without arguments, that works when a property is read,
- `set`: a function with one argument, that is called when the property is set,
- `enumerable`: same as for data properties,
- `configurable`: same as for data properties.

<!--more-->

```javascript
let user = {
  name: "John",
  surname: "Smith",

  get fullName() {
    return `${this.name} ${this.surname}`;
  },

  set fullName(value) {
    [this.name, this.surname] = value.split(" ");
  }

};

alert(user.fullName); // John Smith
```

- We don’t call `user.fullName` as a function, we read it normally.
- e.g. `user.fullName` and `user.fullName = "Wayne Zhang"`

