---
title: 'JS 基础问题 QA'
date: 2021-8-12 13:47:43
categories: FE
tags: [js基础, QA问题]
---

# 数据类型
## null, undefined
- null 是空值，即初始时表示值为空，之后再赋值
- undefined 是 未定义

```js
null + 4 // 4
{} + 4 // 4
undefined + 4 // NaN
[] + 4 // "4"
"" + 4 // "4"
```
<!--more-->

## 布尔值
- 基础数据类型自动转换为`false`:
  ```js
  0
  NaN
  ""
  false
  undefined
  null
  ```
- 空数组和对象: `true`

## 数值
- NaN不等于任何值：`NaN !== NaN`
- `0/0 // NaN`
- Infinity大于一切数值（除了NaN），-Infinity小于一切数值（除了NaN）
- `0 * Infinity // NaN`

### `Number()`
```js
Number([]) // 0
Number([123]) // 123
Number({}) // NaN
Number(null) // 0
Number(undefined) // NaN
```

### `parseInt`
```js
parseInt('+') // NaN
parseInt('+1') // 1
parseInt('0x10') // 以0x开头的16进制：16 
parseInt('011') // 以0开头 忽略掉0 ：11

// 自动转换为科学计数法的会有错误
parseInt(0.0000008) // 8 
parseInt('8e-7') // 等同于：8

// 2-36有效
parseInt('10', 37) // NaN
parseInt('10', 1) // NaN

// 不是数值，会被自动转为一个整数
// 0、undefined和null，则直接忽略
parseInt('10', 0) // 10
parseInt('10', null) // 10

// 第一个参数不是字符串，会被先转为字符串
parseInt(0x11, 2) // 1
parseInt(011, 2) // NaN
// 等同于
parseInt(String(0x11), 2) // 按十六进制
parseInt(String(011), 2) // 按八进制
// 等同于
parseInt('17', 2) // 1
parseInt('9', 2)
```

### `isNaN()`
```js
// 只对数值有效，如果传入其他值，会被先转成数值
isNaN("123") // false

// 空数组 和 只有一个数值成员的数组
isNaN([]) // false
isNaN([123]) // false
isNaN(['123']) // false
isNaN(Number({})) // true
isNaN(Number(['xzy'])) // true
```

## 对象
### 属性
- 所有键名(key/也称为属性property或方法) 都是字符串(也有Symbol), 所以加不加引号都行:
  ```js
  var obj = {
      1: "a",
      3.3: "b",
      <!-- 不符合标识名的条件 -->
      "1p": "必须加引号"
  }

  obj[1] == "a"
  obj["1"] == "a"
  ``` 

- 变量指向对象，是指向了一个内存地址。
  - 不同的变量名指向同一个对象，那么它们都是这个对象的引用
  - 取消某一个变量对于原对象的引用，不会影响到另一个变量
- 点运算符：直接表示字符串属性，不需加引号。
  > ※ 数字属性不能用点运算符 
- 方括号运算符: 需要加引号，否则识别成一个变量
  > `obj["foo"] !== obj[foo]` 
- 删除属性：`delete obj.p // true`
  > 无法删除继承的属性 
  > 只有`configurable: false` 时，返回false

#### 判断是否为自身属性
- `obj.hasOwnProperty('toString')) // false`
- `'toString' in obj // true`
- `for (var p in obj)`
  > 遍历可遍历（enumerable）的属性，跳过不可遍历的属性
  > 自身的和继承的都显示 

## 函数
### 函数定义
- 采用**function命令**声明函数会提升。**赋值语句**定义函数不会
- 具名函数只能在函数内部调用, 赋值后就没有意义
  - `name` 属性会返回具名函数 `function`字段后的名称
  ```js
  var f3 = function myName() {};
  f3.name // 'myName'
  ```

### 函数的作用域
- 作用域与变量一样, 就是其声明时所在的作用域，与其运行时所在的作用域无关。
- 形成**闭包**现象
```js
var a = 1;
var x = function () {
  console.log(a);
};
function f() {
  var a = 2;
  x();
}

f() // 1
```

- 与传入参数不同
```js
var a = 1;
var x = function (a) {
  console.log(a);
};
function f() {
  var a = 2;
  x(a);
}

f() // 2
```

### 参数
- 传入基础类型是**值传递**, 不会改变全局的值
- 传入对象是**地址传递**，更改形参的属性，实参的属性也会变。
  > 如果替换掉整个参数，这时不会影响到原始值
  > ∵ 将形参指向了另一个地址，故不会改变原地址的内容

#### 同名参数
- 取最后出现的那个值。
```js
function f(a, a) {
  console.log(a);
}
f(1) // undefined
```

### arguments
#### callee
- 返回它所对应的原函数

### 闭包
```js
function f1() {
  var n = 999;
  function f2() {
    console.log(n);
  }
  return f2;
}
f1()(); // 999
```

- 闭包就是函数f2: 能够读取其他函数内部变量的函数
- 其两大特点：
  1. 可以读取函数内部的变量另一个就是
  2. 让这些变量始终保持在内存中，即闭包可以使得它诞生环境一直存在
  3. 封装对象的私有属性和私有方法


## 数组
### 不推荐用 `for in`
- 不仅遍历数字属性，还遍历非数字键

### 空位 和 undefined 区别
- `delete` 删除某位后形成空位, 对 `length` 没有影响: `[,,,]`
- 空位都会**被跳过**: `forEach` 方法、`for...in`结构、以及`Object.keys`方法进行遍历
- `undefined`**不会跳过**: `[undefined,undefined,undefined]`

# 运算符：相加
## 布尔值
- 布尔值转换为数值
  1. 俩布尔值相加
  2. 布尔值 和 数值相加

## 字符串
- **从左到右**依次决定是 连接 还是 相加

```js
'3' + 4 + 5 // "345"
3 + 4 + '5' // "75"
```

## 对象
- 必须先转成原始类型的值，然后再相加：`obj + 2 // "[object Object]2"`
  > 是因为调用了 `valueOf`得到对象本身，再调用`toString` 得到 `[object Object]`
  > 特例：如果运算子是一个`Date`对象的实例，那么会优先执行`toString`方法

## 其余运算符 - * /
- 一律转为数值

# 数据类型转换
## Number 同上
> `null`转为数值时为`0`，而`undefined`转为数值时为`NaN`

## String
### 基础类型
- 转换为对应的字符串形式

### 对象
```js
String({a: 1}) // "[object Object]"
String([1, 2, 3]) // "1,2,3"
```

### 自动转化
- 先将复合类型的值转为原始类型的值，再将原始类型的值转为字符串
```js
'5' + {} // "5[object Object]"
'5' + [] // "5"  ∵ [] 为空
'5' + function (){} // "5function (){}"
```

## Boolean
```js
// Boolean() 返回false
false
undefined
null
0   //（包含-0和+0）
NaN
'' // （空字符串）

// 其余都返回true
[]
{}
Boolean(new Boolean(false)) // true
```

# Error
- `new Error(message)` 配合name 和 stack使用

```js
function f() {
  try {
    throw '出错了！';
  } catch(e) {
    console.log('捕捉到内部错误');
    throw e; // 一遇到throw就直接执行finally。这句原本会等到finally结束再执行
  } finally {
    return false; // 直接返回，不会返回去throw e
  }
}
try {
  f();
} catch(e) {
  // 此处不会执行
  console.log('caught outer "bogus"');
}
//  捕捉到内部错误
```

# Object 对象
1. Object对象本身的方法: 被Object直接使用 `Object.print()` 
2. Object的实例方法: 定义在Object原型对象`Object.prototype`上的方法, 被Object的实例直接使用 `var obj = new Object(); obj.print()`
   - obj 是一个实例，继承了`Object.prototype`的属性和方法

## `Object()`函数
- 保证其是Object
  1. null/undefined: 返回空对象
  2. 数/字符串/boolean: `obj instanceof Object // true` + `obj instanceof Number // true`
  3. 对象/数组/函数：不转换，返回原对象/数组/函数
- 区别于 `new Object()` 构造函数

## Object 静态方法
### `Object.keys` 和 `Object.getOwnPropertyNames`
- `Object.keys`方法返回自身的（而不是继承的）所有属性名，只返回可枚举的属性。
- `Object.getOwnPropertyNames`返回对象自身的所有属性名，还返回不可枚举的，例如：length



## Object 实例方法
- `Object.prototype.valueOf()` `Object.prototype.toString()` 等

### `obj.hasOwnProperty`
- 实例对象**自身**是否具有该属性。 `toString // false`

## 判断类型
- `Object.prototype.toString.call(value)`：
  - 函数：返回`"[object Function]"`
  - 数值：返回`"[object Number]"`
  - Error 对象：返回`"[object Error]"`
  - RegExp 对象：返回`"[object RegExp]"`
  - 其他对象：返回`"[object Object]"`

# 数组
- 一般不用 `new Array()` 语法，有歧义
```js
// 单个正整数参数，表示返回的新数组的长度
new Array(1) // [ empty ]
new Array(2) // [ empty x 2 ]

// / 多参数时，所有参数都是返回的新数组的成员
new Array(1, 2) // [1, 2]
new Array('a', 'b', 'c') // ['a', 'b', 'c']
```

## 静态方法
`Array.isArray()`: 弥补 typeof 的不足

# 包装对象
- `new Number()` `new String()` `new Boolean()` 生成的是对象，与原数字/字符串/布尔值 不相等

# Date对象
## 普通函数用法
- 不管有没有参数：返回一个代表当前时间的字符串

## 构造函数
```js
// 参数为时间零点开始计算的毫秒数
new Date(1378218728000)
// 参数为日期字符串
new Date('January 6, 2013');
// 参数为多个整数: 年、月、日、小时、分钟、秒、毫秒
new Date(2013, 0, 1, 0, 0, 0, 0)
// 0表示一月
// 日：1到31。 日期设为0，就代表上个月的最后一天。
// 小时：0到23。
// 分钟：0到59。
// 秒：0到59
// 毫秒：0到999
```

# 面向对象编程
## new 命令
- 原理：
  1. 创建一个空对象，作为将要返回的对象实例。
  2. 将这个空对象的原型，指向构造函数的prototype属性。
  3. 将这个空对象赋值给函数内部的this关键字。
  4. 开始执行构造函数内部的代码。

## `Object.create()` 
- 创建实例对象: 在没有构造函数的情况下，利用现有的对象生成新的实例对象

