---
title: Vue Router
date: 2021-8-18 13:58:22
categories: FE
tags: [vue, vue-router]
---

# VUE Router 入门
## 起步
### `this.$router`
- 利用`this.$router`在任何组件内都可以随时访问路由器

## 动态路由匹配
- 动态路径参数：`'/user/:id'`可以匹配 `/user/foo`、`/user/bar`等
- `this.$route.params`可以访问到参数值： `$route.params.id`
  - 多段路径参数也都存在`$route.params`中
- 通配符：` path: '*'` 放在最后，一般匹配404页面
  - `$route.params` 内会自动添加一个名为 `pathMatch` 参数: `$route.params.pathMatch == /non-existing`

<!--more-->

### 响应参数变化
- 原来的组件实例会被复用，也意味着组件的生命周期钩子不会再被调用。
- 想对路由参数的变化作出响应的话，可以
  - watch `$route` 对象

  ```js
  watch: {
      $route(to, from){...}
  } 
  ```

  - 引入的 `beforeRouteUpdate` 导航守卫

### 示例
```html
<!-- 导航成功后，自动绑定class: `.router-link-active` -->
<p>
    <router-link to="/foo">Fo to Foo</router-link>
    <router-link to="/bar">Fo to Bar</router-link>
</p>

<!-- 路由出口，匹配到的组件显示在这里 -->
<router-view></router-view>
```

```js
import Foo from "./foo.vue"
import Bar from "./bar.vue"

// 每个路由映射一个组件
const routes = [
    {path: "/foo", component: Foo},
    {path: "/bar", component: Bar}
]

const router = new VueRouter({
    routers
})

const app = new Vue({
    router
}).$mount("#app")
```

## 嵌套路由
以 `/user/profile` 和 `/user/posts`为例
- `<router-view>`渲染出口显示最高级路由匹配到的组件，`/user`
  - 一个被渲染组件同样可以包含自己的嵌套 `<router-view>`，并且在`VueRouter`使用参数`children`

  ```js
  const router = new VueRouter({
      routes: [
          {
              path: '/user/:id',
              component: User,
              children: [
                  {
                      path: profile,
                      component: UserProfile
                  },
                  {
                      path: posts,
                      component: UserPosts
                  },
              ]
          }
      ]
  })
  ``` 

## 编程式导航
### `router.push(location, onComplete?, onAbort?)`
- 可以是字符串，也可以是描述地址的对象
  > 提供了 `path` 会忽略掉`params`

```js
// `/user/123`
router.push({ name: 'user', params: { userId }}) 
// `/register?plan=private`
router.push({ path: 'register', query: { plan: 'private' }})
``` 

### `router.replace(location, onComplete?, onAbort?)`
- 不会向 history 添加新记录， 而是替换掉

### `router.go(n)`
- 同 `window.history.go`

## 命名视图 `name`
- 一个视图需要一个组件渲染，多个视图需要明确是哪个组件： 利用 `<router-view name="a"></router-view>`
  > 没有命名的默认为： `name = "default"`

```js
components: {
  default: Foo,
  a: Bar,
  b: Baz
}
```

## 重定向 `redirect`
- `{ path: '/a', redirect: { name: 'foo' }}`
  > 导航守卫并没有应用在跳转路由上，而仅仅应用在其目标上

## 别名 `alias`
- `{ path: '/a', component: A, alias: '/b' }`
  > 当用户访问 /b 时，URL 会保持为 /b，但是路由匹配则为 /a
- 将 UI 结构映射到任意的 URL，而不是受限于配置的嵌套路由结构

## 路由组件传参
### `props` 将组件和路由解耦
```js
const User = {
  props: ['id'],
  template: '<div>User {{ id }}</div>'
}

const router = new VueRouter({
  routes: [
    // id 传入为 组件属性 props
    { path: '/user/:id', component: User, props: true },

    // 对于包含命名视图的路由，你必须分别为每个命名视图添加 `props` 选项：
    {
      path: '/user/:id',
      components: { default: User, sidebar: Sidebar },
      props: { default: true, sidebar: false }
    }
  ]
})
```

### 布尔模式
- 如果 `props` 被设置为 `true`: `route.params` 将会传入为组件属性`props`。

### 对象模式
- `props: { name: "world" }`：静态值

### 函数模式
- `props: route => ({ query: route.query.q })`
  - URL `/search?q=vue` 会将 `{query: 'vue'}` 作为属性传递给 SearchUser 组件。

## 导航守卫 
“导航”表示路由正在发生改变

### 全局前置守卫 `router.beforeEach((to, from, next) => {...})`
- 守卫是异步解析执行，此时导航在所有守卫 resolve 完之前一直处于 **等待中**。
- `next`: 调用该方法来 resolve 这个钩子,  确保一定要调用 `next()`
  - `next()`: 进行管道中的下一个钩子。如果全部钩子执行完了，则导航的状态就是 `confirmed`
  - `next(false)`: 中断当前的导航。如果浏览器的 URL 改变了 (可能是用户手动或者浏览器后退按钮)，那么 URL 地址会重置到 `from` 路由对应的地址
  - `next('/')` or `next({ path: '/' })`: 跳转到一个不同的地址。当前的导航被中断，然后进行一个新的导航
  - `next(error)`: 如果传入 next 的参数是一个 Error 实例，则导航会被终止且该错误会被传递给 `router.onError()` 注册过的回调。

### 全局解析守卫 `router.beforeResolve`
- 导航被确认之前，同时在所有组件内守卫和异步路由组件被解析之后被调用

### 全局后置钩子 `router.afterEach((to, from) => {...})`
- 和守卫不同的是，这些钩子不会接受 next 函数也不会改变导航本身：

### 路由独享的守卫 `beforeEnter`
- 在路由配置上直接定义，与`router.beforeEach((to, from, next) => {...})`一样

```js
routes: [
    {
      path: '/foo',
      component: Foo,
      beforeEnter: (to, from, next) => {
        // ...
      }
    }
  ]
```

### 组件内的守卫
- `beforeRouteEnter`: 对应路由被 confirm 前调用
  > 不！能！获取组件实例 `this`: 因为当守卫执行前，组件实例还没被创建 
  > 可以通过传一个回调给 next来访问组件实例
- `beforeRouteUpdate`: 当前路由改变，但是该组件被复用时调用: 在 `/foo/1` 和 `/foo/2` 之间跳转的时候
- `beforeRouteLeave`: 导航离开该组件的对应路由时调用
  > 用于确认用户在未保存前离开

  ```js
  beforeRouteLeave (to, from, next) {
    const answer = window.confirm('Do you really want to leave? you have unsaved changes!')
    if (answer) {
      next()
    } else {
      next(false)
    }
  }
  ``` 

### 完整的导航解析流程
1. 导航被触发。
2. 在失活的组件里调用 beforeRouteLeave 守卫。
3. 调用全局的 beforeEach 守卫。
4. 在重用的组件里调用 beforeRouteUpdate 守卫
5. 在路由配置里调用 beforeEnter
6. 解析异步路由组件
7. 在被激活的组件里调用 beforeRouteEnter
8. 调用全局的 beforeResolve 守卫
9. 导航被确认。
10. 调用全局的 afterEach 钩子。
11. 触发 DOM 更新。
12. 调用 beforeRouteEnter 守卫中传给 next 的回调函数，创建好的组件实例会作为回调函数的参数传入。

### 导航完成前获取数据
- 在 `beforeRouteEnter` 守卫中获取数据，当数据获取成功后只调用 `next` 方法。
  > 在跳转前获取，应显示些进度条等 

## 滚动行为 `scrollBehavior`
- `{ x: number, y: number }`
- `{ selector: string, offset? : { x: number, y: number }}`

```js
scrollBehavior (to, from, savedPosition) {
  if (savedPosition) {
    return savedPosition
  } else {
    return { x: 0, y: 0 }
  }
}
```

- 滚动到锚点

```js
scrollBehavior (to, from, savedPosition) {
  if (to.hash) {
    return {
      selector: to.hash
    }
  }
}
```

### 平滑滚动 `behavior: 'smooth'`
```js
scrollBehavior (to, from, savedPosition) {
  if (to.hash) {
    return {
      selector: to.hash,
      behavior: 'smooth',
    }
  }
}
```

# 主要API
## 路由对象
- `$route.path`:
  - `String`: 解析为绝对路径，如 `"/foo/bar"`
- `$route.params`:
  - `Object`: 一个 key/value 对象, `{post_id: "123"}`
- `$route.query`:
  - `Object`: 一个 key/value 对象，表示URL的 查询参数。对于路径 `/foo?user=1`，则有 `$route.query.user == 1`
- `$route.hash`:
  - `String`
- `$route.fullPath`
  - `String`: 完成解析后的 URL，包含查询参数和 hash 的完整路径。
- `$route.matched`: 
  - `Array`: 路由记录。当URL 为 `/foo/bar`，将包含从上到下的所有对象
    ``` js
     {
      path: '/foo',
      component: Foo,
      children: [
        // 这也是个路由记录
        { path: 'bar', component: Bar }
      ]
    }
    ```
- `$route.name`:
  - 当前路由的名称，如果有的话。
- `$route.redirectedFrom`:
  - 如果存在重定向，即为重定向来源的路由的名字


