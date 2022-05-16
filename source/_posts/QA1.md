---
title: QA1
date: 2021-8-18 21:20:22
categories: FE
tags: [面试题]
---

# 第一次 QA 记录
## 数字变字符串方法
1. `Number()`
2. `+ -`
3. `parseInt()`

### 字符串转数字
1. `toString()`
2. `+ ""`
    
## || 场景
- `&&`: find the first falsy value; 如果第一个是true，则返回第二个
<!--more-->
 
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

## Promise
### Promise.all
- **顺序不变**
- 一个失败就会返回 error
- **失败仍会执行**，但不监控他们的结果

### Promise.allSettled
- 等所有的都执行完，**不管结果**
  - `{status:"fulfilled", value:result}` for successful responses
  - `{status:"rejected", reason:error}` for errors.

### Promise.race
- 返回最快完成的 (result/error)
- 剩下的忽略

## Object API
### Object.definedProperty(obj, prop, desc) / Object.definedProperties(obj, props)
- configurable
- writable
- enumerable
- value
- get
- set

### Object.getOwnPropertyDescriptor(obj, prop) / Object.getOwnPropertyDescriptors(obj)
- 获取一个对象的**自身属性**的描述符

### Object.getOwnPropertyNames(obj) / Object.getOwnPropertyName(obj, prop)
- 返回指定对象的所有**自身属性名** (包括**不可枚举**属性, 但不包括Symbol值)
  > 只包括**可枚举**属性：`Object.keys()` `for...in`

### Object.getPrototypeOf(obj)
- 返回指定对象的原型

### obj.hasOwnProperty(prop)
- 对象**自身属性**中是否具有指定的属性，忽略原型链上的值
  > `for ... in` 查找对象**自身**或**其原型链中**的属性
  > `Object.keys()` 查找**自身**


### Object.entries
- 返回一个给定对象**自身** **可枚举**属性的键值对数组:  `[ ['foo', 'bar'], ['baz', 42] ]`
```js
for (var [key, val] in Object.entries(obj)) {
    console.log(`${key}: ${val}`)
}

// 效果等于
for (var key in obj) {
    console.log(`${key}: ${obj[key]}`)
}
```

- 还可以方便的转成Map：`new Map(Object.entries(obj))`

### Object.fromEntries
-  把键值对列表转换为一个对象: 把iterable: Map/Array 转为对象


### 总结
- 只有 `for ... in` 是自身 + 继承属性 (可以通过hasOwnProperty去除)
- 只有 `Object.getOwnPropertyName`有不可枚举


## Vue渲染出错
1. VUE
   - `errorHandler`: 指定组件的**渲染和观察期间**未捕获错误的处理函数
   ```js
   Vue.config.errorHandler = function (err, vm, info) {
     // handle error中`info` 是 Vue 特定的错误信息，比如错误所在的生命周期钩子
   ``` 

    - `warnHandler`
    ```js
    Vue.config.warnHandler = function (msg, vm, trace) {
       // `trace` is the component hierarchy trace
     }
    ```

    - `renderError`: 不适用于全局，和组件相关
    ```js
    render (h) {...},
    // 在网页上显示错误信息
    renderError (h, err) { 
        return h('pre', { style: { color: 'red' }}, err.stack)
    }
    ```

    - `errorCaptured`: 捕获一个来自子孙组件的错误, 返回false阻止错误继续向上传播
    ```js
    errorCaptured(err,vm,info) {
      console.log(`cat EC: ${err.toString()}\n info: ${info}`); 
       return false;
    }
    ``` 
[参考](https://blog.csdn.net/Fundebug/article/details/92570054?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-1.control&spm=1001.2101.3001.4242)
[文档](https://vuejs.org/v2/api/#errorHandler)

2. `window.onerror`
   - 可以捕获异步错误，但是：不能捕获资源加载错误错误
   ```js
   window.onerror = function(message, source, lineno, colno, error) {
     post('error', {
       data: {
         error: error.stack
       }
     });
   ``` 

3. `window.addEventListener("error", (e) => {...})`
   - 除了onerror捕获的错误之外，还可以捕获资源加载错误

   ```js
   window.addEventListener('error', (event) => {
     post('error', {
       data: {
         error: event.error.stack
       }
     });
   ``` 
4. `addEventListener("unhandledrejection"，handle`
   - 捕获未catch 的 promise 错误

5. 直接`try...catch`
  - 无法捕获语法、异步错误，但是：可以捕获async await等读数据库的错误

## `for in` 和 `for of`
### `for in` 遍历
- 自身**可枚举**的属性(prop)
- 数组一般使用 `for of`：∵ `for in`不仅遍历数字属性，还遍历非数字键
  > e.g. `arr.otherKey = "asd"` 后，也会把`otherKey`遍历出来

### `for of`
- 在可迭代对象的值(value)

## `Symbol`
### `for ... of`
- 当一个对象实现了`Symbol.iterator`属性时，我们认为它是可迭代的
- ES5之后：`for ... of`调用`Symbol.iterator`实现迭代

## TS 两个不同的合并在一起
- & or ｜

## implements / extends
### implements
- 从父类或者接口实现所有的属性和方法。
- **只能用于类上**，接口不能实现接口或类

### extends
- 继承一个类或接口，继承所有属性和方法。不可以重写属性，但可以重写方法
- **类不可以继承接口**，类只能继承类（成为其子类）
- 接口可以继承接口或类

### 接口 interface
- 多余属性检查，防止使用不属于接口的属性
- 类 implements 接口：需要实现接口的方法和属性 / 接口不能 implements接口
- 接口 extends 类：继承类的成员但不包括其实现，包括`private`和`protected`的话，这个接口只能被这个类或其子类implements
- 接口 extends 接口：复制到接口里，不需要重写 / 类不能 extends 接口
- 类 extends 类：成为其子类

## type
- type 和 interface 很像，但是可以作用于原始值（基本类型），联合类型，元组以及其它任何你需要手写的类型。
- 创建了一个新名字来**引用类型**
- 不能被 `extends`和 `implements`（但可以通过交叉类型`&`来实现）

```ts
// primitive
type Name = string;

// object
type PartialPointX = { x: number; };
type PartialPointY = { y: number; };

// union
type PartialPoint = PartialPointX | PartialPointY;

// tuple
type Data = [number, string];

// dom
let div = document.createElement('div');
type B = typeof div;
```

## proxy
- Proxy 是一个对象，它包装了另一个对象，并允许你拦截对该对象的任何交互。

## 正则
- `/^(http|https)+/`
- 详情见正则总结