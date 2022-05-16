---
title: Vue 响应性
date: 2021-8-31 16:20:22
categories: FE
tags: [vue]
---

# 跟踪变化
## Proxy
- Proxy 是一个对象，它包装了另一个对象，并允许你拦截对该对象的任何交互
- `new Proxy(target, handler)`

### `Reflect`
- 解决this绑定问题：将任何方法都绑定到这个 Proxy
<!--more-->
```js
const dinner = {
  meal: 'tacos'
}

const handler = {
  get(target, property, receiver) {
    return Reflect.get(...arguments)
  }
}

const proxy = new Proxy(dinner, handler)
console.log(proxy.meal) // tacos
```

### 实现响应性
1. 当一个值被读取时进行追踪: proxy 的 `get` 处理函数中 `track` 函数记录了该 property 和当前副作用。
   - `track(target, property)`跟踪一个 property 何时被读取：它将检查当前运行的是哪个副作用，并将其与 target 和 property 记录在一起

2. 当某个值改变时进行检测: 在 proxy 上调用 `set` 处理函数。
   - 当 property 值更改时重新运行这个副作用
    ```js
     const handler = {
         get(target, property, receiver) {
           track(target, property)
           return Reflect.get(...arguments)
         },
         set(target, property, value, receiver) {
           trigger(target, property)
           return Reflect.set(...arguments)
        }
     }
    ``` 
3. 重新运行代码来读取原始值: `trigger` 函数查找哪些副作用依赖于该 property 并执行它们

- `data` 返回的对象将被包裹在响应式代理中，并存储为 `this.$data`。
- Vue 将把 `sum` 的函数包裹在一个副作用中。当我们试图访问 `this.sum` 时，它将运行该副作用来计算数值。
- 包裹 `$data` 的响应式代理将会追踪到，当副作用运行时，property `val1` 和 `val2` 被读取了。

### 渲染响应变化
- 模板被编译成一个 `render` 函数。渲染函数创建 `VNodes`。它被包裹在一个副作用中，允许 Vue 在运行时跟踪被“触达”的 property。
- 类似于一个 `computed` property
- 任何一个 property 发生变化 → 触发副作用再次运行 → 重新运行 render 函数以生成新的 VNodes → 对DOM进行必要的修改

## 响应性基础
### `reactive`
- 为 JavaScript 对象创建响应式状态，是由Proxy包裹
- 推荐复杂的数据类型时使用, 可以不用加value

```js
import { reactive } from 'vue'

// 响应式状态
const state = reactive({
  count: 0
})
```

### `ref`
- 创建独立的响应式值. `ref(0)`返回一个可变的响应式对象, 包含一个名为 value 的 property
- 只有访问嵌套的 ref 时需要在模板中添加 `.value`
  - 作为 property 返回并可以在模板中被访问时：自动解包
  - 作为响应式对象的 property 被访问或更改时：自动解包

```js
import { ref } from 'vue'

const count = ref(0)
console.log(count.value) // 0

count.value++
console.log(count.value) // 1
```

```vue
<template>
    <button @click="count ++">Increment count</button>
    <button @click="nested.count.value ++">Nested Increment count</button>
</template>
```

- 当从 Array 或原生集合类型如 Map访问 ref 时，不会进行解包
```js
const books = reactive([ref('Vue 3 Guide')])
// 这里需要 .value
console.log(books[0].value)

const map = reactive(new Map([['count', ref(0)]]))
// 这里需要 .value
console.log(map.get('count').value)
```

### `toRef`
- 用于为源响应式对象上的属性新建一个ref，从而保持对其源对象属性的响应式连接。
- 接收两个参数：**源响应式对象** 和 **属性名**，返回一个**ref数据**。
- 例如使用父组件传递的props数据时，要引用props的某个属性且要保持响应式连接时。
- 不是原始数据的拷贝，而是引用，改变结果数据的值也会同时改变原始数据

```js
export default defineComponent ({
    props: [title],
    setup(props) {
        const myTitle = toRef(props, "title")
        console.log(myTitle.value)
    }
})
```
### `toRefs`
- 响应式状态解构直接使用会破坏响应，需要转换为一组 ref

```js
const book = reactive({...})
// 转换为一组 ref
let { author, title } = toRefs(book)
// 我们需要使用 .value 作为标题，现在是 ref
title.value = 'Vue 3 Detailed Guide' 
console.log(book.title) // 'Vue 3 Detailed Guide'
```

### `isRef`
- 检查一个值是否为一个 ref 对象

### `unref`
- `val = isRef(val) ? val.value : val` 的语法糖

### watchEffect
- 立即执行传入的一个函数，同时响应式追踪其依赖，并在其依赖变更时重新运行该函数

#### 停止监听
- 放在 `setup()` 函数或 生命周期钩子 被调用时, 在组件卸载时自动停止
- 可以显式调用返回值以停止侦听 
```js
const stop = watchEffect(() => {...})
// later
stop()
```

#### 清除副作用 `onInvalidate`
- 副作用函数会执行一些异步的副作用，这些响应需要在失效时清楚。`watchEffect` 接收一个 `onInvalidate` 函数作入参，注册清理失效时的回调

```js
const data = ref(null)
watchEffect(async onInvalidate => {
  onInvalidate(() => {
    /* ... */
  }) // 我们在Promise解析之前注册清除函数
  data.value = await fetchData(props.id)
})
```

#### 侦听器调试 `onTrack` 和 `onTrigger`
- `onTrack` 将在响应式 property 或 ref 作为依赖项被追踪时被调用。
- `onTrigger` 将在依赖项变更导致副作用被触发时被调用。



### watch
- 多个同步更改只会触发一次侦听器。但是可以用 `await nextTick()`等待侦听器在下一步改变之前运行


## Vue2.x以下
### 声明property
- 不允许动态添加根级响应式 property，必须实例前声明所有根级响应式，哪怕是空值

### 异步更新队列
- 基于更新后的 DOM，利用`Vue.nextTick(callback)`确保DOM更新完成后被调用

# 原理
- `data` 收集依赖：`Object.defineProperty()` 设置getter/setter
- 观察变化 `watch`：watch
- 

```js
class Dep {
    constructor() {
        this.subscribers = [];
    }
    store() {
        if( target && !this.subscribers.includes(target) ) {
            this.subscribers.push(target)
        }
    }
    notify() {
        this.subscribers.forEach(sub => sub());
    }
}

const dep = new Dep()
let price = 10;
let number = 2

function watcher (myFunc) {
    target = myFunc;
    dep.store();  // 每次先收集每个变量依赖
    target();     // 执行运算得到total
    target = null; 
}

watcher(() => {
    total = price * number;
})
```

```js
let data = {
    price: 5,
    number: 2
}
Object.keys(data).forEach( (key) => {
    let internalVal = data[key];
    
    Object.defineProperty(data, key, {
        get() {
            console.log(`${key} is ${internalVal}`);
            return internalVal;
        },
        set(newVal) {
            console.log(`Set ${key} to ${newVal}`);
            internalVal = newVal;
        }
    })
})
```

- 结合
```js
let target, total;
Object.keys(data).forEach( (key) => {
    let internalVal = data[key];
    const dep = new Dep();
    
    Object.defineProperty(data, key, {
        get() {
            dep.store()
            return internalVal;
        },
        set(newVal) {
            internalVal = newVal;
            dep.notify()
        }
    })
})

function watcher (myFunc) {
    target = myFunc;
    target();     // 执行运算得到total
    target = null; 
}

watcher(() => {
    total = price * number;
})
```

## 利用 defineProperty的缺点
- 对属性的操作，使用这种方法无法拦截：给对象新增属性
- 改成Proxy进行数据劫持

## computed 和 watch区别
### Computed
- 支持缓存。只有依赖的数据发生了变化，才会重新计算
- 不支持异步
- computed的值会默认走缓存，基于data声明过，或者父组件传递过来的props中的数据进行计算的。如果一个属性是由其他属性计算而来的，这个属性依赖其他的属性，一般会使用computed
- 如果computed属性的属性值是函数，那么默认使用get方法，函数的返回值就是属性的属性值；在computed中，属性有一个get方法和一个set方法，当数据发生变化时，会调用set方法。
 
 ### Watch：
- 不支持缓存，数据变化时，它就会触发相应的操作
- 支持异步监听，访问API
- 监听的函数接收两个参数，第一个参数是最新的值，第二个是变化之前的值
- 当一个属性发生变化时，就需要执行相应的操作
- 监听数据必须是data中声明的或者父组件传递过来的props中的数据，当发生变化时，会出大其他操作，函数有两个的参数：
    - immediate：组件加载立即触发回调函数
    - deep：深度监听，发现数据内部的变化，在复杂数据类型中使用，例如数组中的对象发生变化。需要注意的是，deep无法监听到数组和对象内部的变化。

## Computed 和 Methods 的区别
- 将同一函数定义为Computed 和 Methods，对于最终的结果，两种方式是相同的
- 不同点：
  - computed: 计算属性是基于它们的依赖进行缓存的,只有在它的相关依赖发生改变时才会重新求值对于 method ，只要发生重新渲染，
  - methods: 调用总会执行该函数