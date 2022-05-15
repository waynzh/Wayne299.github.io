---
title: 'Basic and Data types'
date: 2020-8-29 19:20:43
categories: FE
tags: [js基础, The-modern-javascript-tutorial]
---

# Basic JavaScript
## Comparison
### null and undefined
- `null == undefined // true` 除此之外他们和其他任何只都是false
- `+null = 0`
- `+undefined = NaN`

### null vs 0
``` javascript
null > 0   // (1) false
null == 0  // (2) false
null >= 0  // (3) true
```
<!--more-->

- `==`equality 和 `> < >= <=`comparisons不同，comparisons将`null`转化为0。

## Logical Operator
- `&&`: find the first falsy value; 如果第一个是true，则返回第二个
 
``` javascript
1 && 0  // return 0
2 && 3  // return 3 
```
  
- `||`: find the first truthy value; 都是false返回最后一个

``` javascript
1 || 0  // return 1 (后一个不执行)
null || 2  // return 
  
```

- `??`:  select the first “defined” variable. 
- Forbidden to use it together with `&&` and `||` operators, except use explicit parentheses.
 
```javascript
a ?? b  // a, if it’s not null or undefined; b, otherwise.
(a !== null && a !== undefined) ? a : b;
```
 
- e.g. 

``` javascript
function showCount(count) {
	alert(count ?? "unknown");
}
  
showCount(0); // 0
showCount(); // unknown
``` 

## Object
### A primitive as an object
#### Objects are “heavier” than primitives
- `user.sayHi()` is “heavier” than `str.toUpperCase()`

#### Always truthy

```javascript 
let zero = new Number(0);

if (zero) { // zero is true, because it's an object
  alert( "zero is truthy!?!" );
}
```


### Computed Properties
- We can use square brackets in an object literal, when creating an object.
  
 ```javascript
	let fruit = prompt("Which fruit to buy?", "apple");
	let bag = {
		[fruit]: 5, // the name of the property is taken from the variable fruit
	};
	
	bag.apple // 5 if fruit="apple"
 ```

### Property value shorthanded

```javascript
let user = {
  name,  // same as name:name
  age: 30
};
```

### Property existence test, `in` operator

```javascript
let user = { name: "John", age: 30 };

"age" in user    // true, with quotes means property; without quotes means variable
"blabla" in user // false
```

### `for...in` loop
- `for (key in object) {}`: Integer properties are sorted, others appear in creation order. 

### Store in references
- When an object variable is copied – the reference is copied, the object is not duplicated. 
- So we have two variables, each one with the reference to the same object:

```javascript
let user = { name: 'John' };
let admin = user;

admin.name = 'Pete'; // changed by the "admin" reference
alert(user.name); // 'Pete', changes are seen from the "user" reference
```

### Comparions by reference
```javascript
let a = {};
let b = a; // copy the reference

alert( a == b ); // true, both variables reference the same object
alert( a === b ); // true
```

```javascript
let a = {};
let b = {}; // two independent objects

alert( a == b ); // false
```

### Clone and merge
####  `Object.assign()`, shallow copy
`Object.assign(dest, [src1, src2, src3...])`

```javascript
let user = {
  name: "John",
  age: 30
};

let clone = Object.assign({}, user);
```
#### Nest cloning (deep cloning)
- Use the cloning loop that examines each value of `user[key]` and, if it’s an object, then replicate its structure as well.
- deep cloning function: `_.cloneDeep(obj)` from lodash.


## Symbol 
### `Symbol()`
- Symbols are created with `Symbol()` call with an optional description (name).
- Symbols are guaranteed to be unique.
- Don't auto-convert to a string

```javascript
// id1/id2 is a symbol with the description "id"
let id1 = Symbol("id");
let id2 = Symbol("id");

alert(id1 == id2); // false, unique even if with the same description
```

### “Hidden” object properties
- If `user` objects belongs to another code, and that code also works with them, we shouldn’t just add any fields to it. `let id = Symbol("id"); user[id] = "Our id value";`
- Donot participate in `for..in` loop and `Object.keys(user)`.
- But `Object.assign` copies both string and symbol properties:

### Global symbol
####`Symbol.for(name)`
- Store in global symbol registry.

```javascript
// read from the global registry
let id = Symbol.for("id"); // if the symbol did not exist, it is created

// read it again (maybe from another part of the code)
let idAgain = Symbol.for("id");

// the same symbol
alert( id === idAgain ); // true

```

#### Symbol.keyFor(sym)

```javascript
// get symbol by name
let sym = Symbol.for("name");
let sym2 = Symbol.for("id");

// get name by symbol
alert( Symbol.keyFor(sym) ); // name
alert( Symbol.keyFor(sym2) ); // id
```

## Numbers
-  in hex (0x), octal (0o) and binary (0b) systems.

### `toString(base)`

```javascript
let num = 255;

alert( num.toString(16) );  // ff
alert( num.toString(2) );   // 11111111

// double dot to call a method directly on a number
alert( 123456..toString(36) ); // 2n9c 
```

### Imprecise calculations
- `0.1 + 0.2 !== 0.3` solution:
  - `toFixed(n)`: `sum.toFixed(2) // "0.30" ` -> string convenient to `"$0.30"` / or coerce it into a number use the unary plus `+`.

### Finite & NaN
- `isNaN("1") // false` 
- `NaN === NaN // false`  
- `Object.is`: SameValue.
  - `Object.is(NaN, NaN) === true`
  - `Object.is(0, -0) === false`

### parseInt & parseFloat
- `parseInt(str, base);`

```javascript
alert( parseInt('100px') ); // 100
alert( parseFloat('12.5em') ); // 12.5

alert( parseInt('12.3') ); // 12
alert( parseFloat('12.3.4') ); // 12.3

alert( parseInt('a123') ); // NaN, "a" stops the process.
```


## String
### `charAt(index)`

```javascript
let str = `Hello`;

alert( str[1000] ); // undefined
alert( str.charAt(1000) ); // '' (an empty string)
```

### includes, startsWith, endsWith -- ES6
- `str.includes(substr, pos)`: returns `true / false`.
- `str.startsWith(substr, pos)`: returns `true / false`.
- `str.endsWith(substr, strlength)`: default `strlength = str.length`.

### str.indexOf() -- old version
- `str.lastIndexOf(substr, position)`: reverse version.
- `if(~str.indexOf("id"))`: bitwise `~n = -(n+1)`.


```javascript
let str = 'Widget with id';

str.indexOf('Widget') // 0, because 'Widget' is found at the beginning
str.indexOf('widget') // -1, not found
str.indexOf("id")     // 1, "id" is found at the position 1 

str.indexOf('id', 2)  // 12
```

### substring, slice, substr
- `str.slice( start[, end) )` 
  * 返回一个新的数组，不改变原数组
  * Negative values are availible. `str.slice(-4, -1)  // [-4, -1)` 
  * `arr.slice()` 创造一个一样的新数组。
  
- `str.substring( start[, end) )`
  * Allows `start` to be greater than `end`. 
  * Not allows negative values, negative values mean `0`.
  
- `substr(start, length)`: -- old version
  * Allows negative start.

### `for...of`
- iterate over characters.

```javascript
for (let char of "Hello") {
  alert(char); //  "H", then "e", then "l" etc
}
```

### immutable
- `str[0] = 'h'; // error`


## 数组 Array
- `for...of`: its value, e.g. `for item of items`
- `for...in`: its index, e.g. `for key in arr`.  -- never use in Array, but Object
  * It iterates over all properties, not only the numeric ones.

### Add & delete
#### 首位插入： `unshift`
#### 首位删除： `shift`
#### 末尾插入： `push`
#### 末尾删除： `pop`

#### 数组合并： `concat`

#### `splice(index, howmany, [insertItem)`
- 改变原数组
- 返回值是删除的数组
- Negative indexes allowed. 

```javascript
let arr = [1, 2, 5]; 
arr.splice(-1, 0, 3, 4); // replace the index -1 with 3, 4

alert( arr ); // 1,2,3,4,5
```
 
#### `slice(start[, end)` 
- 返回选择的·新·的数组，不改变原数组。
- Negative values are availible.

### Searching
#### `arr.some(func)` / `arr.every(func)`
- callback boolean value if pass the func.

#### `indexOf(subArr)` / `lastIndexOf(subArr)` / `includes(subArr)`

```javascript
const arr = [NaN];
arr.indexOf(NaN)  // -1 (should be 0, but === equality doesn't work for NaN)
arr.includes(NaN) // true (correct)
```

#### `find(func)` / `findIndex(func)`

```javascript
let result = arr.find(function(item, index, array) {
  // if true, item is returned and iteration is stopped (findIndex returns index
  // If nothing found, returns undefined (findIndex returns -1
});
```

#### `filter(func)`
- 返回所有true的item

```javascript
let results = arr.filter(function(item, index, array) {
  // if true item is pushed to results and the iteration continues
  // returns empty array if nothing found
});
```

### Transform
#### `sort(func)`
- if `arr.sort()` without func, the array elements are converted to strings, then sorted according to each character's Unicode code point value.

- `compareFunction(a, b)`:
  * 小于0 ==>   `a` 会被排列到 `b` 之前
  * 等于0 ==>   `a`  `b` 相对位置不变
  * 大于0 ==>   `b` 会被排列到 `a` 之前

- which means:
  * `return -1`: a < b; `negative num`: less
  * `return 0`: a = b;
  * `return 1`: b < a;  `positive num`: greater
  
  

```javascript
sort( (a, b) => return a-b; )  // Ascending order
```

- 具体理解:
  1. 当`a<b`时，`a-b<0`。即`return 一个负值`。根据compareFunction 的行为定义，a 会被排列到 b 之前，是升序排列方式。

  2. 当`a>b`时，`a-b>0`。即`return 一个正值`。根据compareFunction 的行为定义，b 会被排列到 a之前，也是升序排列方式。

  3. 当a=b时，a-b=0。即return 零。根据compareFunction 的行为定义，a 和 b 的相对位置不变。

#### `map(func)`
- iterate and return the data for each element.

```javascript
let result = arr.map(function(item, index, array) {
  // returns the new value instead of item
});
```

#### `reduce(func[,initialValue)` , `reduceRight`
- calculate a single value based on the array.
- 没有initialValue，`curr = arr[1]`开始遍历,  `accumulator = arr[0]`; EXTREME CARE!!  If the array is empty, then reduce call without initial value gives an error.

```javascript
let value = arr.reduce(function(accumulator, curr, index, array) {
  // ...
}, [initialValue]);
```

- 作用
  * 二维数组变一维
  * 数组所有值求和
  * 去重

### Destructuring assignment(ES6)
#### Syntax
- `[first, last] = iterable`
- Ignore elements using commas `,`
- and the rest of the array items is also skipped.

```javascript
// we have an array with the name and lastname
let arr = ["Wayne", "ignore", "Zhang", "something", "doesn't", "matter" ]

// sets firstName = arr[0]
// and lastName = arr[2]
let [firstName, , lastName] = arr;

alert(firstName); // Wayne
alert(lastName);  // Zhang
```

#### Works fine with any iterable(right) and anything(left)

```javascript
let [a, b, c] = "abc"; // ["a", "b", "c"]

let user = {};
let [user[1], user[2], user[3] ] = new Set([1, 2, 3]);
```

#### Looping with `entries()`
- `for ( let [key, value] of Object.entries(user) )`

#### Swap variables trick (ES6)
- `[firstName, lastName] = [lastName, firstName]`  

#### The rest `...`
- that type of `rest` is Array.

```javascript
let [name1, name2, ...rest] = ["Julius", "Caesar", "Consul", "Wayne"];

alert(name1); // Julius
alert(name2); // Caesar

// Note that type of `rest` is Array.
alert(rest[0]); // Consul
alert(rest[1]); // Wayne
alert(rest.length); // 2
```

#### Default valuem `=`
- `let [name = "Guest", lastName = "Anonymous"] = ["Wayne"]`
-  Default value or function will run only for the value that not provided.

### Check type
#### `Array.isArray(arr)`
- `Array.isArray( {} ); // false   Array.isArray( [] )  // true` 
- `typeof {}; // object   typeof []  // object` not Array.


## Arrow Function
### `value => expr` & `value => {...}`
- Arrow function will treat `{...}` as the start of function body, not the start of the object. So when we mean object, we need to use `( {object..} )`

```javascript
let usersMapped = users.map(user => ({         // as a object
  fullName: `${user.name} ${user.surname}`,
  id: user.id
}));
```

## Iterables
- `for...of`: its value, e.g. `for item of items`
- `for...in`: its index, e.g. `for key in arr`.  -- never use in Array, but Object

### `Symbol.iterator`
- When `for..of` starts, it calls `Symbol.iterator` method once (or errors if not found).
- The method **must return an iterator** – an object with the method `next()`. When it wants the next value, it calls next() on that object.
- The result of `next()` **must have the form** `{done: Boolean, value: any}`, where `done=true` means that the iteration is finished, otherwise value is the next value.

```javascript
let range = {  // iterable, but not array-like
	from: 1,
	to: 5
};

range[Symbol.iterator] = function() {
	return {
		curr = this.from,
		last = this.to,
		
		next() {  // must return an iterator with the method next().
			// must have the form{done: Boolean, value:any}
			if(this.curr <= this.last) {
				return {done: false, value: this.curr+=1 }
			} else {
				return {done: true};
			}
		}
	}
}

for(let num of range) alert(num)  // works fine with for...of
```

### Iterator explicite

```javascript
let iterator = str[Symbol.iterator](); // create the iterator.

// we can split the iteration process
while (true) {
  let result = iterator.next(); 
  if (result.done) break;
  alert(result.value); // outputs characters one by one
}
```

### `Array.from(obj, mapFn, thisArg)`

```javascript 
// iterable: `range` above.
// array-like: Objects that have indexed properties and length, but lack the built-in methods of arrays.
let arrayLike = {  
  0: "Hello",
  1: "World",
  length: 2
};

let arr = Array.from(arrayLike); 
alert(arr.pop());  // method works
```



## Map
- `map.set(key,value)`
- `map.get(key)`
- `map.has(key)`
- `map.delete(key)`
- `map.clear()` / `map.size`

```
let map = new Map( [    // array of [key,value] pair
  ['cucumber', 500],
  ['tomatoes', 350],
  ['onion',    50]
] );
```

- `Map` allows **keys of any type**: string / numeric / boolean / object keys; But `Object` would convert keys to string.
- `map[key]` isn’t the right way to use a `Map`. We shold use methods: `set`, `get` ...

### Compare keys in `Map`
- `Map` uses `SameValueZero(x,y)`: roughly the same as `===`, but `NaN` equals `NaN`.

### Chaining
- `map.set`: return the map itself. `map.set("1","str1").set(1,"num1");`

### Iteration
- the same order as the values were inserted
- `map.keys()`: returns an **iterable** for `keys`, but not an array.
- `map.values()`: returns an **iterable** for `values`, but not an array.
- `map.entries()`: returns an iterable for entries`[key, value]`, default in `for...of`
- `map.forEach( (value, key, map) => {...} )`

``` javascript
// iterate over keys 
for (let vegetable of recipeMap.keys()) {
  alert(vegetable); // cucumber, tomatoes, onion
}

// iterate over values 
for (let amount of recipeMap.values()) {
  alert(amount); // 500, 350, 50
}

// iterate over [key, value] entries
for (let entry of recipeMap) { // the same as of recipeMap.entries()
  alert(entry); // cucumber,500 (and so on)
}
```

### Map from Object, `Object.entries(obj)`
- `let map = new Map(Object.entries(obj));`

### Object from Map, `Object.fromEntries(arr)`
- `let obj = Object.fromEntries(map.entries());` or `Object.fromEntries(map)`
- Usually pass it to a 3rd-party code that expects a plain object.

## 集合 Set
- `set.has(value)`
- `set.add(value)`
- `set.delete(value)` or `set.remove(value)`
- `set.size` / `set.clear()`

- `set.keys()` = `set.values()` returns an **iterable** object for `values`, but not an array.
- `set.entries()`: returns an iterable object for entries.

* 只有key(without value) / 或者key 和 value值一样
* 原生`Set` add读取的是地址， 地址不同值相同的也可以添加在一起
* hasOwnProperty / delete

### Order
- `Set` and `Map` is always in the insertion order.
- But we can’t reorder elements or directly get an element by its number.

## WeakMap
- It serves as an additional storage. But not for an arbitrary data

### Garbage collection
- Properties of an object or elements of an array or another data structure are considered reachable while that data structure is in memory: 

```javascript
let john = { name: "John" };
let array = [ john ];
map.set(john, "...");

john = null;

// john is stored inside the array and map
map.keys(); // john = { name: "John" }
array[0];
```

### Keys MUST be `Object`
- `weakMap.set(obj, "fine")` 
- if there is no other reference to `obj`, it'll be removed from memorary.
- Not support iteration and methods `keys()`, `values()` `entries()`


## Object
### `Object.keys(obj)`, `values`, `entries`
#### Difference between `Object` and `Map`
- `map.keys()`: returns **iterable**.
- `Object.keys(obj)`: returns **Array**.
- `Object.*` method returns real array.

#### Ignore `Symbol` properties
- like `for...in` loop, ignore properties that use `symbol` as keys.
- `Object.getOwnPropertySymbol()`: returns array of only symbolic keys.
- `Reflect.ownKeys(obj)` that returns all keys.

#### Transforming objects
1. `Object.entries(obj)` to get an array of `[key,value]` pair. e.g. [ [],[] ]
2. use array methods. e.g. `map`
3. `Object.fromEntries(array)` turn it back into an object

### Object destructuring
- `let {var1, var2} = {var1:…, var2:…}`

```javascript
let options = {
  title: "Menu",
  width: 100,
  height: 200
};

// changed the order in let {...}
let {height, width, title} = { title: "Menu", height: 200, width: 100 }
alert(title);  // Menu
alert(width);  // 100
alert(height); // 200

// { sourceProperty: targetVariable }
// {what: goes where}
let {width: w, height: h, title="Default"} = options;
alert(title);  // Menu
alert(w);      // 100
alert(h);      // 200s
```

### Using existing variables, without `let` (Warning!!)

```javascript
let title, width, height;

// error in this line
{title, width, height} = {title: "Menu", width: 200, height: 100};

// wrap in parentheses ()
( {title, width, height} = {title: "Menu", width: 200, height: 100} );
```

- Treats `{}` as the code block. Add parentheses `(...)` to show it's not a code block.

### Examples
- When it comes to many parameters. We can pass parameters as an object, it's clear and well-documented.

```javascript
let options = {
  title: "My menu",
  items: ["Item1", "Item2"]
};

function showMenu({     // pass parameters as an object
  title = "Untitled",
  width: w = 100,  // width goes to w
  height: h = 200, // height goes to h
  items: [item1, item2] // items first element goes to item1, second to item2
}) {...}

showMenu(options);
```

- If we want all values by default

```javascript
showMenu({});    // all values are default
showMenu();      // error
```

- Fix this by making `{}` the default value for the whole object of parameters. `func ( {obj} = {} )`

```javascript
function showMenu({ title = "Menu", width = 100, height = 200 } = {}) {
	// code here
}
```

## JSON 
- Property name and value must be double quotes.
- no `new` is allowed, only bare values.
- no comments.

### `JSON.stringify(value, replacer, space)`
- MUST no circular references.

#### replacer
- If we pass an array of properties to it, only these properties will be encoded.

```javascript
let room = {
  number: 23
};

let meetup = {
  title: "Conference",
  participants: [{name: "John"}, {name: "Alice"}],
  place: room // meetup references room
};

room.occupiedBy = meetup; // room references meetup

alert( JSON.stringify(meetup, ['title', 'participants', 'place', 'name', 'number']) );  // needs all parameters in the list, e.g. `name` `number`
/*
{
  "title":"Conference",
  "participants":[{"name":"John"},{"name":"Alice"}],
  "place":{"number":23}
}
*/
```

#### space (formatting)
- `space = 2` tells JavaScript to show nested objects on multiple lines, with indentation of 2 spaces inside the object.

### `JSON.parse(value, reviver)`
#### reviver
- It works for nested objects as well.

```javascript
let schedule = `{
  "meetups": [
    {"title":"Conference","date":"2017-11-30T12:00:00.000Z"},
    {"title":"Birthday","date":"2017-04-18T12:00:00.000Z"}
  ]
}`;

schedule = JSON.parse(schedule, function(key, value) {
  if (key == 'date') return new Date(value);
  return value;
});

alert( schedule.meetups[1].date.getDate() ); // works!
```
