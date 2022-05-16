---
title: 'Promises & async/await'
date: 2020-9-7 19:20:43
categories: FE
tags: [js基础, The-modern-javascript-tutorial, 面试题, 总结]
---

#  Promise
## 异步操作
- 回调函数：
  - fs 文件操作
  - 数据库操作
  - AJAX
  - 定时器
- Promise：语法上是构造函数 / 功能上使用promise对象 封装一个异步操作并可获取成功或失败的结果值
  - 优点：
    1. 支持**链式调用**，解决回调地狱问题（不便于阅读+不便于异常处理）
      - ∵`.then`方法返回的还是一个`Promise`对象 
    2. **制定回调函数**的方式更加灵活（旧的只能在启动异步任务前制定），但promise是先启动异步任务→返回`promise`对象→给`promise`对象绑定回调函数（甚至可以在异步任务结束后制定多个）
- **队列任务优先级**：`process.nextTick` > `promise`的回调 > `async` > `setTimeout` > `setInterval` > `setImmediate` > I/O > UI rendering

##  Syntax
- 封装一个异步函数 → 成功调`resolve`，失败调`reject` → `.then`方法制定成功和失败的回调，并对结果进行处理
<!--more-->

```javascript
// resolve 和 reject都是函数类型的数据
// 成功 将 promise 对象的状态设置为 【成功】
// 失败 将 promise 对象的状态设置为 【失败】
let promise = new Promise(function(resolve, reject) {
  // executor (the producing code, `resolve()` / `reject()`)
});
```

- callbacks: There can be **only one** result or one error.
  1. `resolve(value)`: if success, with result `value`
  2. `reject(error)`: if error, `error` is the error object.

- internal properties属性:
  1. 状态`PromiseState`: 只能从`pending`到 settled。 且只能改变一次。
    * `pending`: initially
    * `fulfilled`: when `resolve` is called.
    * `rejected`: when `reject` is called.
  2. 对象的值`PromiseResult`: 保存着异步任务【成功/失败】的值。
    * `undefined`: initially
    * `value`: `resolve(value)` called.
    * `error`: `reject(error)` called.


##  handlers: `then`, `catch`, `finally`
- If a promise is returned, then JavaScript waits for it to settle and calls next handlers with its result.

###  `.then(result => {}, error => {})`
#### `util.promisify()` 返回一个promise的版本

```js
const util = require('util');
const fs = require('fs');
// 返回一个新的函数，是一个promise的对象
let mineReadFile = util.promisify(fs.readFile);

mineReadFile('content.txt').then(value => {
  console.log(value.toString());
}, error => {
  console.log(error);
});

// 等于 封装函数读取文件内容，返回promise对象
function mineReadFile(path) {
  return new Promise((resolve, reject) => {
    require('fs').readFile(path, (err, data) => {
      if(err) reject(err);
      resolve(data);
    });
  });
}
```

###  `.catch(f)`
- 执行`reject`的回调：the same as `.then(null, f)`
- **异常穿透**：可以在链式最后指定失败的回调，可以catch前面任何操作的异常

###  `.finally(f)`
- the same as `.then(f,f)`
- good handler for performing cleanup.


##  Promises chaining
- The `value` (`return value` or `resolve(value)`) that first `.then` returns is passed to the next `.then` handler.
- Because `promise.then` returns a promise, so we can call the next `.then` on it.

###  Situation
* if returns a value, `newPromise` gets resolved with the `returned value` as its value;
* if doesn't return anything, `newPromise` gets resolved with an `undefined` value;
* throws an error, `newPromise` gets rejected with the `thrown error` as its value;

### 中断Promise链
- 返回`pending`状态：`return new Promise(() => {});` 后续`.then`将不执行


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
- 返回一个新的Promise：只有所有都成功状态才成功，只要一个失败就直接失败；值就是数组中的结果值/失败就是失败的结果值
- e.g. download several URLs in parallel and process the content once they are all done.

###  syntax
`let promise = Promise.all([...promises...])` 
- 包含n个promise的数组, not just `[]`, any iterable is fine.
- If any of those objects is not a `promise`, it’s passed to the resulting array “as is” (not change).

- **顺序不变**：the order of the resulting array members is the same as in its source promises.
- **失败仍会执行**：If any of the promises is rejected, the promise returned by `Promise.all` immediately rejects with that error. But the others will **still continue** to execute, but `Promise.all` won’t watch them anymore, and their results will be ignored.

### Trick
- map an array of rough data into an array of promises, and wrap that into `Promise.all()`
  * e.g. `url = [...]; request = url.map(url => fetch(url)); Promise.all(request)`


##  Promise.allSettled
- `Promise.all` is good for “all or nothing” cases, but for `Promise.allSettled` just waits for all promises to settle, regardless of the result.

###  result array
- `{status:"fulfilled", value:result}` for successful responses
- `{status:"rejected", reason:error}` for errors.


##  Promise.race
- 返回**最快完成**的：Waits only for the **first(fastest) settled promise** and gets its result (or error).
- 剩下的忽略：all further results/errors are ignored.

### syntax
`let promise = Promise.race(iterable)`


## `Promise.resolve/reject` -- old version
- `Promise.resolve`
  - 传入参数为 非Promise类型的对象，则返回 【成功的Promise对象】
  - 传入参数为 Promise类型的对象，则传入参数的结果决定 【resolve返回的结果】
  - 例如：`let p = Promise.resolve(521);`
- `Promise.reject`
  - 状态永远【失败】，且传入参数是什么就结果就是什么
- e.g. thenables and functions we want write `loadCached(url).then(...)`, and guarantee to return a `promise`.



# Microtasks
## Microtasks queue
- Promise handlers `.then` `.catch` `.finally` are always asynchronous.
- 改变状态后，回调函数都会执行：So even when a `Promise` is immediately resolved, the code on the lines below `.then` `.catch` `.finally` will still execute before these handlers.

>> That's because:
>>  - Execution of a task in this queue is initiated only when nothing else is running.
>>  - So when a promise is ready, its handlers are put into the queue; they are not executed yet. When the JavaScript engine becomes free from the current code, it takes a task from the queue and executes it.


## 改变Promise状态 和 指定回调函数/执行回调函数的顺序
### 状态改变 / 指定回调函数
1. Promise包含【异步操作】，先指定回调函数 → 改变状态
2. Promise【正常操作】，先改变状态 → 指定回调函数

### 改变状态 / 执行回调函数
- 只有状态改变后，才执行



# Async/await
## Async functions, `async`
- `async function xxx` before a function 返回一个 `Promise`对象（用`Promise.resolve()`方法）.
  - `return` 一个 非Promise类型的数据：【状态】成功 【值】return的值
  - `return` 一个 Promise对象：【状态】同 【值】同
  - `throw` 抛出异常: 【状态】失败 【值】异常


## Await, `await`
- works ONLY inside `async` functions: 

```js
async function main() {
  let value = await <Promise>;
}
```

- `await <Promise>`: **wait** until that promise settles and returns its result.
  - 返回Promise成功的值
  - 失败的话就会抛出异常，通过`try...catch`获取
- `await <other>`: 如果表达式是其他值，**不等待**直接返回，返回值就是其本身。

```javascript
async function f() {
  let promise = new Promise((resolve, reject) => {
    setTimeout(() => resolve("done!"), 1000)
  });

  let result = await promise; // wait until the promise resolves (*)
  alert(result); // 1s "done!"
}

f();
```

>> It’s just a more elegant syntax of getting the promise result than `promise.then`, easier to read and write.

### async、promise、setTimeout 混用的执行顺序
- `await`会阻塞其所在表达式中后续表达式的执行

```js
let x = 0;
async function test() {
    x = (await 2) + x;// 把await放在x前面
    console.log(x);	 // 3
}

test();
x = 1;
```

- **队列任务优先级**：`promise.Trick()` > `promise`的回调 > `async` > `setTimeout` > `setImmediate`

```js
        async function async1() {
            console.log("2");
            await  async2();    // 等于 async2() + 阻塞后续执行
            console.log("7");    // 异步2
        }
        async function async2() {
           console.log( '3');   // 相当于return了undefined
        }

        console.log("1");
        setTimeout(function () {
            console.log("8");   // 异步3
        },0);
        async1();
        new Promise(function (resolve) {
            console.log("4");
            resolve();
        }).then(function () {
            console.log("6");      // 异步1
        });
        console.log('5');
        // 1 2 3 4 5 6 7 8
        // Google V8 6和7相反是引擎底层修改
```

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

# Promise自定义封装

```js
function Promise(executor) {
  this.PromiseState = "pending";
  this.PromiseResult = null;
  const self = this; 

  // 保存指定后的回调函数
  this.callbacks = [];

  function resolve(data) {
    // 判断状态是否修改过: ∵只能修改一次
    if(self.PromiseState !== "pending") return;
    // 1. 修改对象状态 （PromiseState）
    self.PromiseState = "fulfilled";
    // 2. 设置对象结果值（promiseResult）
    self.PromiseResult = data;

    // 执行回调函数
    // if(self.callback.onResolved) {
    //   self.callback.onResolved(data);
    // }

    // 异步执行回调函数：状态改变之后不能马上执行
    setTimeout(() => {
      self.callbacks.forEach(item => {
        item.onResolved(data);
      })
    });
    
  }

  function reject(data) {
    if(self.PromiseState !== "pending") return;
    self.PromiseState = "rejected";
    self.PromiseResult = data;

    setTimeout(() => {
      self.callbacks.forEach(item => {
        item.onRejected(data);
      });
    })
  }

  try{
    executor(resolve, reject);
  } catch(e) {
    // 接受报错
    reject(e);
  }
}

// 添加 then 方法
Promise.prototype.then = function(onResolved, onRejected) {
  const self = this;

  // 判断回调函数参数 → 异常穿透
  if(typeof onRejected !== 'function') {
    onRejected = reason => {
      throw reason;
    }
  }
  // 判断回调函数参数 → 没有传递参数
  if(typeof onResolved !== 'function') {
    onResolved = value => value;
  }

  return new Promise((resolve, reject) => {
    // 封装函数
    function callback(type) {
      try{
        // 获取回调函数执行结果 → 同步修改then返回的结果
          let result = type(self.PromiseResult); // 函数内部的this需要换成self
          if(result instanceof Promise) {
            result.then(v => {
              resolve(v);
            }, e => {
              reject(e);
            });
          } else {
            // 结果的状态保存为「成功」+ 值不变
            resolve(result);
          }
        } catch(e) {
          reject(e);
        }
    }

    if(this.PromiseState === "fulfilled") {
      // 异步执行回调函数: 状态改变之后不能马上执行
      setTimeout(() => {
        callback(onResolved);
      });
    } 
    
    if(this.PromiseState === "rejected") {
      // 异步执行回调函数
      setTimeout(() => {
        callback(onRejected);
      });
    }
    // 判断 pending 状态：处理异步任务
    if(this.PromiseState === "pending") {
      this.callbacks.push({
        // 执行成功的回调函数 → 异步修改then返回的结果
        onResolved: function() {
          callback(onResolved);
        },
        onRejected: function() {
          callback(OnRejected);
        }
      });
    }
  })
}

// 添加 catch 方法
Promise.prototype.catch = function(onRejected) {
  return this.then(undefined, onRejected);
}

// 添加 resolve 方法: 属于函数对象 不是实例对象的
Promise.resolve = function(value) {
  return new Promise((resolve, reject) => {
    if (value instanceof Promise) {
      value.then(v => {
        resolve(v);
      }, e => {
        reject(e)
      });
    } else {
      resolve(value);
    }
  });
}

// 添加 reject 方法
Promise.reject = function(reason) {
  return new Promise((resolve, reject) => {
    reject(reason);
  });
}

// 添加 all 方法
Promise.all = function(promises) {
  return new Promise((resolve, reject) => {
    let count = 0;
    let arr = [];

    for(let i = 0; i < promises.length; i++) {
      promises[i].then(v => {
        // 运行到这里即表示 第i个对象成功
        // 但是不能立即执行resolve
        count += 1;
        // 将当前Promise对象的结果存储到arr数组中
        arr[i] = v;
        // 判断全部成功
        if(count === promises.length) {
          // 数组中存储的是所有成功的结果
          resolve(arr);
        }
      }, e => {
        // 只要有error就返回rejected状态
        reject(e);
      })
    }
  });
}

// 添加 race 方法
Promise.race = function(promises) {
  return new Promise((resolve, reject) => {
    for(let i = 0 ; i < promises.length; i++) {
      promises[i].then(v => {
        // 不需要等，谁先完成谁改变状态，并且状态只能改变一次
        resolve(v)
      }, e => {
        reject(e)
      });
    }
  }) 
}
```

# Promise自定义封装 —— class类

```js
class Promise {

  // 构造方法
  constructor(executor) {
    this.PromiseState = "pending";
    this.PromiseResult = null;
    const self = this; 

    this.callbacks = [];

    function resolve(data) {
      if(self.PromiseState !== "pending") return;
      self.PromiseState = "fulfilled";
      self.PromiseResult = data;

      setTimeout(() => {
        self.callbacks.forEach(item => {
          item.onResolved(data);
        })
      });
    }

    function reject(data) {
      if(self.PromiseState !== "pending") return;
      self.PromiseState = "rejected";
      self.PromiseResult = data;

      setTimeout(() => {
        self.callbacks.forEach(item => {
          item.onRejected(data);
        });
      })
    }

    try{
      executor(resolve, reject);
    } catch(e) {
      reject(e);
    }
  }

  // then 方法
  then(onResolved, onRejected) {
    // ...
  }

  catch(onResolved, onRejected) {
    // ...
  }

  // resolve 方法: 属于类，不属于实例对象
  static resolve(value) {
    // ...
  }

  static race(value) {
    // ...
  }
}
```

# 面试题 判断超时

```js
/**
 * promiseTimeout: 利用Promise来进行超时监听
 * @param1 : promise
 * @param2 : time 超时时限
 * @return : 1. 超时 则返回一个reject的promise， reason是new Error...
 *           2. 没有超时 则返回promise
 */
var promiseTimeout = function (promise, time) {
    let rej = new Promise((resolve, reject) => {
        setTimeout(() => {
            reject(new Error("超时！！"));
        }, time);
    });

    return Promise.race([promise, rej]);
};

// 测试
var p1 = new Promise((res, rej) => {
    setTimeout(() => {
        res("p1 resolve!");
    }, 1000);
});

var p2 = new Promise((res, rej) => {
    setTimeout(() => {
        res("p2 resolve!");
    }, 3000);
});

// 通过
promiseTimeout(p1, 2000).then(
    (v) => {
        console.log(v);
    },
    (e) => {
        console.log(e);
    }
);

// 超时
promiseTimeout(p2, 2000).then(
    (v) => {
        console.log(v);
    },
    (e) => {
        console.log(e);
    }
);
```
[示范](https://www.jianshu.com/p/736022b6faec)

# 面试题 实现ajax

```js
function ajax(params) {
    return new Promise((resolve, reject) => {
        var xhr = new XHLHttpRequest();
        xhr.open(params.type, params.url);
        xhr.onreadystatechange = () => {
            if(xhr.readyState == 4) {
                if(xhr.status == 200) {
                    resolve(xhr.responseText);
                } else {
                    reject(xhr.responseText);
                }
            }
        }
        xhr.send();
        
    });
}
```

# 面试题 红绿灯

```js
var d = new Promise((res, rej) => res());
var step = function (def) {
    def.then(() => {
        return tic(red, 3000);
    })
        .then(() => {
            return tic(green, 1000);
        })
        .then(() => {
            return tic(yellow, 2000);
        })
        .then(() => {
            step(def);
        });
};

step(d);
```

# queue(list, count)

```js
function fn1(callback) {
    setTimeout(() => {
        console.log(1);
        callback();
    });
}
function fn2(callback) {
    setTimeout(() => {
        console.log(2);
        callback();
    });
}
function fn3(callback) {
    setTimeout(() => {
        console.log(3);
        callback();
    });
}

function myFunc(list, count) {
    if (list.length == 1) return list[0](null);
    list = [list[count - 1], ...list.slice(0, count - 1).concat(list.slice(count))];

    var p = new Promise((resolve) => {
        resolve();
    });

    for (let i = 0; i < list.length; i++) {
        p.then((v) => {
            list[i](list[i + 1]);
        });
    }
}

myFunc([fn1, fn2, fn3], 2);
```















