---
title: Vuex
date: 2021-8-18 14:21:13
categories: FE
tags: [vue, vuex]
---

# Vuex
## 开始
```js
import Vue from "vue";
import Vuex from "vuex"

Vue.use(Vuex)

const store = new Vuex.Store({
    state: {
        count: 0
    },
    mutation: {
        increment: {
            state.count++
        }
    }
})
```

- 通过操作`mutation` 来改变`state`:
  - 通过 `store.state`获取状态，`store.commit('increment')`触发状态变更
<!--more-->
- 在Vue实例中，也要提供创建好的`store`, 方便组件中访问 `this.$store property`

```js
new Vue({
  el: '#app',
  store: store,
  methods: {
    increment() {
      this.$store.commit('increment')
      console.log(this.$store.state.count)
    }
  }
})
```

## State
### `mapState`
```js
template: `<div>{{ count }}</div>`,
computed: {
  count () {
    return this.$store.state.count
  }
}
```

- 为了反复申明`computed`，使用展开符和mapState统一添加

```js
import { mapState } from 'vuex'

export default {
    computed: {
        localComputed() {...}
        ...mapState({
            count: state => state.count,
            // 传字符串等同于 state => state.count
            countAlias: "count",
            // 为了正确使用 `this.localCount`, 必须用常规函数形式
            countPlusLocalCount (state) {
                return this.localCount + state.count
            }
        })
    }
}
```

- 当映射的 `computed` 的名称与 `state` 的子节点名称相同时，我们也可以传一个字符串数组。

```js
computed: mapState([
  // 映射 this.count 为 store.state.count
  'count',
  'increment'
])
```

## Getter
- 类似于store 的计算属性
- 第一个参数接受`state` 第二个接受其他`getters`

```js
getters: {
  doneTodos: state => {
    return state.todos.filter(todo => todo.done)
  },
  doneTodosCount: (state, getters) => {
    return getters.doneTodos.length
  }
}
```

- 通过`this.$store.getters.doneTodosCount`在组件中使用它
  > 通过**属性访问**时每次都会去进行调用，而不会缓存结果
  > 也不能直接当作 方法 来使用，因为只是一个属性

### 通过方法访问
- 让getter返回一个函数，来作为方法传参: `store.getters.getTodoById(2)`

```js
getters: {
    getTodoById: state => id => {
        return state.todos.find(todo => todo.id === id)
    }
}
```

### `mapGetters`
- 辅助将 store 中的 getter 映射到局部计算属性

```js
import {mapGetters} from "vuex"

export default {
    // ...
    computed: {
        ...mapGetters({
            'doneTodosCount',
            'anotherGetter',
            // this.toThisName 映射到 this.$store.getters.changeName
            toThisName: 'changeName'
        })
    }
}
```

## Mutations
- 类似于事件注册，真正的事件更改应该用 `store.commit("event")`

### commit payload
```js
mutations: {
    increment(state, payload) {
        state.count += payload.amount
    }
}
// ...
store.commit("increment", {
    amount: 10
})
```

### commit对象 风格
```js
store.commit({
    type: 'increment',
    amount: 10
})
```

### 添加新属性时
- 使用 `Vue.set(obj, 'newProp', 123)`, 或者 新的替换老的`state.obj = { ...state.obj, newProp: 123 }`

### Mutations 必须是同步函数
- 实际上，任何在回调函数中进行的状态的改变都是不可追踪的。

### 组件中提交Mutation: `mapMutations`
```js
import {mapMutations} from "vuex"

export default {
    // ...
    methods: {
        ...mapMutations([
            "increment",
            "incrementBy"
        ]),
        // 或者 将 `this.add()` 映射到 `this.$store.commit("increment")`
        ...mapMutation({
            add: "increment"
        })
    }
}
```

## Actions
- 类似于Mutations，不同在于：
  - 处理异步操作
  - 提交的是Mutation，而不是直接更改状态

```js
const store = new Vuex.Store({
    state: {
        count: 0
    },
    mutations: {
        increment(state) {
            state.count++
        }
    },
    actions: {
        increment(context) {
            context.commit("increment")
        }
    }
})
```

- Action函数 接受一个 `context`, 与 `store` 实例具有相同方法和属性
  - `context.state` 和 `context.getters` 获得`state` 和 `getters`
  - `context.commit()` 提交一个`mutation`

### 解构语法
```js
actions: {
    increment( {commit} ) {
        commit("increment")
    }
}
```

### 分发Actions: `store.dispatch`
- 为了实现异步操作, 通过`store.dispatch`触发Action

```js
actions: {
    incrementAsync ({commit}) {
        setTimeout(() => {
            commit("increment")
        }, 1000)
    }
}
```

```js
// dispatch payload 形式
store.dispatch("incrementAsync", {
    amount: 10
})

// dispatch对象 形式
store.dispatch({
    type: "incrementAsync",
    amount: 10
})
```

### 购物车事例
- 通过提交 mutation 来记录action产生的状态变更。

```js
actions: {
    checkout({commit, state}, products) {
        const saveCartItems = [...state.cart.added]
        commit(types.CHECKOUT_REQUEST)
        shop.buyProducts(
            products,
            // 成功操作
            () => commit(types.CHECKOUT_SUCCESS),
            // 失败操作
            () => commit(types.CHECKOUT_FAILURE, saveCartItems)
        )
    }
}
```

### 在组件中分发 Action: `mapActions`
- `this.$store.dispatch('xxx')` 或者 `mapActions` 辅助函数将组件的 methods 映射为dispatch调用

### 组合Actions
- dispatch 可以处理被触发的Action 所返回的Promise, `store.dispatch`返回 `Promise`

```js
actions; {
    actionA ( {commit} ) {
        return new Promise((res, rej) => {
            setTimeout(() => {
                commit("someMutation")
                res()
            }, 1000)
        })
    },
    actionB ({dispatch, commit}) {
        // 可以使用 then语法
        return dispatch("actionA").then(() => {
            commit("someOtherMutation")
        })
    }
}
```

#### async / await
```js
actions: {
  // getData() 和 getOtherData() 返回的是 Promise
  async actionA({commit}) {
      commit("gotData", await gotData())
  },
  async actionB({dispatch, commit}) {
      // 等待actionA 完成
      await dispatch("actionA") 
      commit("gotOtherData", await gotOtherData())

      // 类似于
      await dispatch('actionA')
      const response = await getOtherData()
      commit('gotOtherData', response)
  }
}
```

## Modules
- 模块有自己的 `state` `mutations` `actions` `getters`, 或者嵌套模块 `modules`

```js
// 分割 module
const moduleA = {
    state: () => {...},
    mutations: {},
    actions: {},
    getters: {}
};

const moduleB = {
    state: () => {...},
    mutations: {},
    actions: {},
    getters: {}
};

// 分配 module
const store = new Vuex.Store({
    modules: {
        a: moduleA,
        b: moduleB,
    }
})

// 调用
store.state.a // → moduleA的 state
```

### 模块内的局部状态
- 对于模块内的 `getters` 和 `mutations`:模块内的局部状态 `state` 是第一个参数
- 对于模块内 `actions`: 获取局部状态通过 `context.state`；根结点 `context.rootState`
  ```js
  actions: {
    incrementIfOddOnRootSum ({ state, commit, rootState }) {
      if ((state.count + rootState.count) % 2 === 1) {
        commit('increment')
      }
    }
  ``` 
- 对于模块内的 `getters`: 根节点状态会作为第三个参数暴露出来
  ```js
  getters: {
    sumWithRootCount (state, getters, rootState) {
      return state.count + rootState.count
    }
  }
  ``` 
  
### 命名空间 `namespaced: true`
- 在`modules`中，可以命名各个module，并添加 `namespaced: true`

```js
const store = new Vuex.Store({
    modules: {
        // 命名module
        account: {
            namespaced: true,
            state: () => ({...}), // 函数申明不怕引用被共享
            getters: {
                isAdmin() {...} // getters["account/isAdmin"]
            },
            actions: {
                login() {...} // dispatch("account/login")
            },
            mutations: {
                login() {...} // commit("acount/login")
            }
        }
    }
})
```

#### 带命名空间的模块 访问 全局内容
- 使用全局内容
  1. getters: `someGetter (state, getters, rootState, rootGetters) {...}`
  2. actions: `someAction ({ dispatch, commit, getters, rootGetters }) {...}` or `context.rootGetters`
- 分发全局内容: 将 `{ root: true }` 作为第三参数
  1.  `dispatch('someOtherAction', null, { root: true }) // -> 'someOtherAction'`  而不是 `'account/someOtherAction'`
  2.  `commit('someMutation', null, { root: true }) // -> 'someMutation'`

#### 带命名空间的模块 注册 全局action
- 添加`root: true`, 并将其定义放在 `handle`中

```js
// modules中 并且 namesapced: true
actions: {
    callAction: {
        root: true,
        handler (namespacedContext, payload) {
            let {state, commit} = namespacedContext;
            commit('setText');
            alert(state.text);
        }
    }
}
```

#### 带命名空间的 绑定函数
- 可以将模块的**空间名称字符串**作为第一参数传递给 `mapState`, `mapGetters`, `mapActions` 和 `mapMutations`

```js
computed: {
    ...mapState("some/nested/module", {
        a: state => state.a
    })
},
methods: {
    ...mapActions("some/nexted/module", [
        "foo", // this.foo()
        "bar"
    ])
}
```

- 或者利用`createNamespacedHelpers`创建基于某个命名空间辅助函数

```js
import { createNamespacedHelpers } from 'vuex'
const { mapState } = createNamespacedHelpers('some/nested/module')

export default {
  computed: {
    // 在 `some/nested/module` 中查找
    ...mapState({
      a: state => state.a,
      b: state => state.b
    })
  }
```

### 模块动态注册 `store.registerModule`
```js
const store = new Vuex.Store({ ... })

// 注册模块 myModule
store.registerModule("myModule", {...})
// 注册嵌套模块 nested/myModule
store.registerModule(['nested', 'myModule'])

// 调用
store.state.myModule
store.state.nested.myModule
```
#### 保留state `preserveState: true`
- 假设 store 的 state 已经包含了这个 module 的 state , 并且不希望将其覆写
- 注册一个新module时，保留过去的state: `store.registerModule('a', module, { preserveState: true })`

### 模块重用
- 场景：
  1. 多个store公用同一模块
  2. 在一个store中，模块被多次注册 
- 多次引用 state 就会通过引用被共享，导致状态对象被修改时 store 或模块间数据互相污染的问题。
- 类似于 data 的问题，所以用 函数申明 代替 对象申明

```js
const reusableModule = {
    state: () => ({
        foo: "bar"
    })
}
```

