---
title: Vue Intro
date: 2021-8-19 16:52:12
categories: FE
tags: [vue, 面试题, 总结]
---

# VUE 入门 —— 极客时间

# 生命周期钩子
- `created` `mounted` `updated` 和 `unmounted`
[生命周期图示](https://vue3js.cn/docs/zh/images/lifecycle.png)

> ※ 不要在选项 property 或回调上使用箭头函数, 因为没有this

## Vue基础语法
## 模版语法
<!--more-->
- 只可以包含单个 表达式, 如
  - `{ { number + 1 } } `
  - `{ { ok ? 'YES' : 'NO' } }` 三元可以，if不行 
  - `{ { message.split('').reverse().join('')} }`
  - `<div v-bind:id="'list-' + id"></div>`

## 2.x: `new Vue`  3.x: `vue.createApp`
- 实例对象: 有改动都会改动
- vm: ViewModel 的缩写, 表示组件实例。

```js
const RootComponent = { /* 选项 */ }
const app = Vue.createApp(RootComponent) // 根组件：渲染的起点
const vm = app.mount('#app')             // Vue应用挂载到vm上，传递 #app
                                        // 返回的是根组件实例
```

## 响应式
- 只有当实例被创建时就已经存在于 `data` 中的 property 才是响应式的
- `Obj.freeze(obj)` 阻止修改现有的 property

## 指令
### 动态参数
`<a v-bind:[attributeName]="url"> ... </a>`
- 参数为 JavaScript 表达式：`[attributeName]`
> 动态参数的约束
>> 不能用 空格 和 引号：报错['foo' + bar]
>> 避免使用大写字符来命名键名: [someAttr]会被转换成[someattr]


### `v-bind:`
- 动态绑定变量
- 缩写`:id`

```html
<div :id="message"></div>

<script>
    var vm = new Vue({
        el: '#app',
        data: {
            message: "hello world"
        }
    })
</script>
```

### `v-if` `v-else` `v-else-if`
- 做判断，不满足即不挂载到DOM上

```html
<span v-if=“item.del”>{ {item.title} }</span>
<span v-else style="text-decoration: line-through">{ {item.title} }</span>

<script>
    var vm = new Vue({
        el: '#app',
        data: {
            item: {
                title: "课程",
                del: false
            }
        }
    })
</script>
```

### `v-show` 区别于`v-if`
- `v-show="!item.del"` 做判断 切换元素的 `display` property。
  - `true`即显示
  - `false`即`display:none` 但总是会被渲染, 挂载在DOM上

### `v-for`
```html
<li v-for="item in list">
    <span v-if=“item.del”>{ {item.title} }</span>
    <span v-else style="text-decoration: line-through">{ {item.title} }</span>
</li>

<script>
    var vm = new Vue({
        el: '#app',
        data: {
            list: [{
                title: "课程1",
                del: false
            }, {
                title: "课程2",
                del: false
            }]
        }
    })
</script>
```

- 可以结合`computed`，返回过滤或排序后的数组:
  1. `v-for="n in func"`
  2. `v-for="n in func(val)"`

```html
<li v-for="n in evenNumbers">{ { n } }</li>
```

```js
data() {
  return {
    numbers: [ 1, 2, 3, 4, 5 ]
  }
},
computed: {
  evenNumbers() {
    return this.numbers.filter(number => number % 2 === 0)
  }
}
```

> 不推荐 `v-if` 和 `v-for`一同使用
>>  `v-if`优先级更高，所以访问不到`v-for`里的变量
>> 可以把`v-for`上移，里面嵌套`v-if`

### `v-on` 
- 操作逻辑，不操作DOM
- 接收方法名称，缩写`@click="reverseMessage"`
- 访问原始的 DOM 事件。可以用特殊变量 `$event` 传入

```html
<button v-on:click="reverseMessage">反转 Message</button>

<script>
    var vm = new Vue({
        el: '#app',
        data: {
            message: 'Hello Vue.js!'
        },
        methods: {
            reverseMessage() {
                this.message = this.message
                  .split('')
                  .reverse()
                  .join('')
            }
        }
    })
</script>
```


#### 事件修饰符
- `.stop` : `event.stopPropagation()`
- `.prevent` : `event.preventDefault()`
- `.capture` :
- `.self` :
- `.once` :
- `.passive` : 遵循默认行为，即不阻止默认行为
  - 不把 `.passive` 和 `.prevent` 一起使用，因为 `.prevent` 将会被忽略

#### 按键修饰符
- 可以将任意有效按键名转换为 kebab-case 来作为修饰符

```html
<!-- 只有在 `key` 是 `Enter` 时调用 `vm.submit()` -->
<input @keyup.enter="submit" />
```

#### 系统修饰健
- 仅在**按下**相应按键时,才触发鼠标或键盘事件的监听器

```html
<!-- Alt + Enter -->
<input @keyup.alt.enter="clear" />

<!-- Ctrl + Click -->
<div @click.ctrl="doSomething">Do something</div>
```

#### `.exact` 修饰符
- **有且只有**按下相应的才触发: `@click.ctrl.exact`

#### 鼠标按钮修饰符
- 限制处理函数仅响应特定的鼠标按钮: `.left` `.right` `.middle`


### `v-model` 双向绑定
#### 表单输入绑定
- `text` 和 `textarea` 元素使用 `value` property 和 `input` 事件；
- `checkbox` 和 `radio` 使用 `checked` property 和 `change` 事件；
- `select` 字段将 `value` 作为 prop 并将 `change` 作为事件。

```html
<div id="two-way-binding">
    <p>{ { message } }</p>
    <input v-model="message" />
</div>
```

#### 修饰符
1. `lazy` 修饰符: 转为在 `change` 事件之后进行同步
  - `<input v-model.lazy="msg" />`
2. `.number`: 将用户的输入值转为数值类型
   - `<input v-model.number="age" type="number" />` 
3. `.trim`: 过滤用户输入的首尾空白字符 
   - `<input v-model.trim="msg" />` 


###  `v-once`
- 执行一次性地插值，当数据改变时，插值处的内容不会更新。
> ※ 会影响到该节点上的其它数据绑定

### `v-html`
- 使`{ {模版} }`内输出真正的 HTML, 而不是解释为普通文本

```js
<p>Using v-html directive: <span v-html="rawHtml"></span></p>

const RenderHtmlApp = {
  data() {
    return {
      rawHtml: '<span style="color: red">This should be red.</span>'
    }
  }
}
```


# 组件
## 语法
```js
Vue.component("todo-list", {
    props: {
        title: String,
        del: {
            type: Boolean,
            default: false
        }
    },
    template: `
        <li v-for=“item in list">
            <span v-if=“del”>{ {title} }</span>
            <span v-else style="text-decoration: line-through">{ {title} }</span>
        </li>
    `,
    data: function() {
        return {}
    },
    methods: {} 

})
```

## `data` Property
- 组件的`data`是一个**函数**, 所以每个实例都是独立拷贝
- 以 `$data` 的形式存储在组件实例中: `vm.count` 或者`vm.$data.count`

```js
const app = Vue.createApp({
  data() {
    return { count: 4 }
  }
})
```



## `methods`
- 组件的`methods`是一个包含所需方法的**对象**

```js
const app = Vue.createApp({
  methods: {
    increment() {
      // `this` 指向该组件实例
      this.count++
    }
  }
})
```

## `computed` 计算属性
- `computed`只在**相关响应式依赖**发生改变时它们才会重新求值
  - 不同：直接调用方法也可以用 `<p>{ { calculateBooksMessage() } }</p>` 
  - 不同：`methods`每当触发重新渲染时，总会再次执行函数
  
```js
<span>{ { publishedBooksMessage } }</span>

Vue.createApp({
  data() {
    return {}
  },
  computed: {
    // 计算属性的 getter
    publishedBooksMessage() {
      // `this` points to the vm instance
      return this.author.books.length > 0 ? 'Yes' : 'No'
    }
  }
})
```

- 当 `vm.author.books` 发生改变时，所有依赖 `vm.publishedBookMessage` 绑定也会更新
- 可以设置getter (默认) 和 setter (需要时)

```js
computed: {
  fullName: {
    // getter
    get() {
      return this.firstName + ' ' + this.lastName
    },
    // setter
    set(newValue) {
      const names = newValue.split(' ')
      this.firstName = names[0]
      this.lastName = names[names.length - 1]
    }
  }
}
```

## `watch` 侦听器
- 在数据变化时执行异步或开销较大的操作  

### `handler` `immediate`
- 当值第一次绑定的时候，不会执行监听函数。只有值发生**改变**才会执行
- `immediate： true`：第一次绑定时也执行函数
  
```js
watch: {
  firstName: {
    // 在watch中声明就会调用
    immediate: true,
    handler(newName, oldName) {
      this.fullName = this.newName + this.lastName;
    }
  }
}
```

### `deep` 深度监听
- 普通的`watch`方法无法监听到对象内部属性的改变，只监测对象的引用是否改变，比如给obj赋值的时候它才会监听到
- 但 `deep: true`会影响性能，所以可以用**字符串**形式监听对象属性

```js
watch: {
  "obj.a": {
    deep: true,
    immediate: true,
    handle(newVal, oldVal) {
      console.log("obj.a changed.")
    }
  }
}
```



## Class
### 对象语法
1. 动态切换class

```html
<div
  class="static"
  :class="{ active: isActive, 'text-danger': hasError }"
></div>

<div :class="classObject"></div>
```

2. 直接绑定一个数据对象，可以不内联到模板里

```html
<div :class="classObject"></div>
```

```js
classObject: {
  active: true,
  'text-danger': false
}
```

3. 绑定一个返回对象的`computed`
```js
data() {
  return {
    isActive: true,
    error: null
  }
},
computed: {
  classObject() {
    return {
      active: this.isActive && !this.error,
      'text-danger': this.error && this.error.type === 'fatal'
    }
  }
}
```

### 数组语法

```html
<div :class="[isActive ? activeClass : '', errorClass]"></div>
<!-- 条件判断繁琐时 加入对象语法 -->
<div :class="[{ active: isActive }, errorClass]"></div>
```

### 组件中的使用
- 当组件有很多根元素时，利用`$attrs`属性就可以指定哪个部分接受后加的类

```js
const app = Vue.createApp({})

app.component('my-component', {
  template: `
    <p :class="$attrs.class">Hi!</p>
    <span>This is a child component</span>
  `
})
```




## Style
### 对象语法
1. js对象
   ```html
   <div :style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
   ```

2. 直接绑定样式对象，更清晰
   ```html
   <div :style="styleObject"></div>
   ```

   ```js
   data() {
     return {
       styleObject: {
         color: 'red',
         fontSize: '13px'
       }
     }
   }
   ```

### 数组对象

```html
<div :style="[baseStyles, overridingStyles]"></div>
```

#### 多重值
- 只会渲染数组中最后一个被浏览器支持的值

```html
<div :style="{ display: ['-webkit-box', '-ms-flexbox', 'flex'] }"></div>
```

## 通过`prop`向子组件传递数据
- 接受一个单独的 `po` prop,

```html
<blog-post
  v-for="post in posts"
  v-bind:key="post.id"
  v-bind:po="post"
></blog-post>
```

```js
Vue.component('blog-post', {
  props: ['po'],
  template: `
    <div class="blog-post">
      <h3>{ { po.title } }</h3>
      <div v-html="po.content"></div>
    </div>
  `
})
```


## 监听子组件事件 向父组建通讯
### `$emit(String: eventName, [...args])`
- 触发当前实例上的事件。附加参数都会传给监听器回调。
- 父级组件监听这个事件时
  1. 可以通过 `$event` 访问到被抛出的这个值
  2. 事件处理函数是一个方法，则第一个参数就是这个值 

#### 例1 
```html
<my-component @welcome="sayHi"></my-component>
```

```js
const app = Vue.createApp({
    methods: {
        sayHi() {
            console.log("Hi")
        }
    }
})


app.component("my-component", {
    template: `
        <button @click=“$emit("welcome")”> Click me! </button>
    `
})
```

#### 例2 利用额外的参数
```html
<div id="emit-example-argument">
  <advice-component @give-advice="showAdvice"></advice-component>
</div>
```

```js
new Vue({
  el: "emit-example-argument",
  methods: {
    showAdvice: function(advice) {  // 参数advice： 利用额外参数 adviceText
      alert(advice);
    }
  }
})

Vue.component("advice-component", {
  data: function() {
    return {
      adviceText: "Some advice"
    }
  },
  template: `
    <div>
      <input type="text" v-model="adviceText">
      <button @click="$emit('advice', adviceText)">
        Click me for sending advice
      </button>
    </div>
  `
})
```

### 在组件上使用`v-model` ??
```html
<custom-input v-model="searchText"></custom-input>
```

```js
Vue.component("custom-input", {
  props: [modelValue],
  template: `
    <input 
      :value="modelValue"
      @input="$emit("input", $event.target.value)"
    ></input>
  `
})
```

## `<slot>` 插槽
- 向一个组件传递内容, 内容就在`<slot></slot>`之中

```html
<alert-box> Something bad happened!! </alert-box>
```

```js
Vue.component("alert-box", {
  template: `
    <div class="demo-alert-box">
      <strong>Error!</strong>
      <slot></slot>
    </div>
  `
})
```

### 作用域
- slot和模版内部一样能访问到相同的property，但不能访问模版的作用域。例如：

```html
<navigation-link url="/profile">
  Logged in as { { user.name } }
  // 属于模版的作用域 url 是访问不到的: undefined
  Send you to : { {url} }
</navigation-link>
```

### placeholder
- 在 `<slot> placeholder </slot>`中插入placeholder，在使用模版且不提供任何插槽内容时显示

### `name` 具名插槽
```html
<!-- base-layout 组件 -->
<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <!-- 不带 name 的 <slot> 会带有隐含的名字“default” -->
    <slot></slot>
  </main>
```

```js
<base-layout>
  <template v-slot:header>
   <h1>This is for header</h1>
  </template>
  // 其余没有被template包裹的都会被视为默认插槽
  <h1>Other is for main</h1>
  <h1>And another one</h1>

  // 或者明确点 v-slot:default
  <template v-slot:default>
    <h1>Other is for main</h1>
    <h1>And another one</h1>
  </template>
</base-layout>
```

### 插槽prop：作用域
- 想在父组件上自定义时，将所需作为attribute传入 `<slot>`

```html
<!-- <todo-list> template -->
<ul>
  <li v-for="(item, index) in items">
    { { item } }
  </li>
</ul>

<!-- 转换成 -->
<ul>
  <li v-for="(item, index) in items">
    // 在父组建上自定义时，将所需作为attribute传入slot
    <slot :item="item" :index="index"></slot>
  </li>
</ul>
```

- 使用带值的 `v-slot` 定义插槽prop 的名字: 包含所有插入的prop

```html
<todo-list>
  <template v-slot:default="slotProps">
    <span>{ {slotProps.index} }: { {slotProps.item} }</span>
  </template>  
</todo-list>
```


### 只有默认插槽缩写
- 可以省去`<template>`, 直接把`v-slot`用在组件上
- 但只要有多个插槽，就要使w完整的`<template>`语法

```html
<todo-list v-slot="slotProps">
  <span>{ {slotProps.item} }</span>
</todo-list>
```

### 解构插槽Prop
- 可以利用解构赋值：将 属性/值 从 对象/数组 中**逐个对应**取出,赋值给其他变量。
  - 例如：`[a, b, ...rest] = [10, 20, 30, 40, 50]` 中，`rest == [30, 40, 50]`  
  - 例如：`{a,...rest} = {b:20, a:20,c:30,d:40}`中，`rest == { {b: 20, c: 30, d: 40} }`
  - 给新的变量名赋值 且 带有默认值： `{p: foo = 0, q: bar = false} = {p: 42, q: true}`中，`foo == 42` `bar == true`

```html
<current-user v-slot="{ user }"> // 解构
  { { user.firstName } }
</current-user>

<!-- 重命名 -->
<current-user v-slot="{ user:person }">
  { { person.firstName } }
</current-user>

<!-- placeholder -->
<current-user v-slot="{ user = { firstName: 'Guest' } }">
  { { user.firstName } }
</current-user>
```

### 动态插槽名
`<template v-slot:[dynamicSlotName]>`

### 具名插槽的缩写: `#`
- `v-slot:header` 缩写为 `#header`


## 动态组件
### `is`属性 
- 通过 Vue 的 `<component>` 元素加一个特殊的 `is` attribute, 实现组件的动态切换

```html
`<component :is="currentTabComponent"></component>`
```

### `keep-alive`
- 来回的切换动态标签，每次都生成一个新的实例，所以不会继续展示之前选择的内容
- `keep-alive` 可以使组件实例能够被在它们第一次被创建的时候缓存下来

```html
<keep-alive>
  <component v-bind:is="currentTabComponent"></component>
</keep-alive>
```

> 要求被切换到的组件都有自己的名字

## 解析 DOM 模板时的注意事项
- `<ul>` `<ol>` `<table>` 和 `<select>`，对于哪些元素可以出现在其内部有严格限制。
- 直接将组件放入内部会被视为 无效 并 提升到外部。但该限制不存在于：
  1. 字符串 (例如：template: '...')
  2. 单文件组件 (.vue)
  3. `<script type="text/x-template">`
- 其他情况可以通过 `is` 来解决

```html
<table>
  <tr is="blog-post-row"></tr>
</table>
```


## 全局注册 / 局部注册
### 全局注册
- 通过 `Vue.component` 来创建的组件
  - 可以用在任何新创建的 Vue 根实例
  - 在各自组件内部也可以使用

### 局部注册
```js
var ComponentA = { /* ... */ }
var ComponentB = { /* ... */ }
// 在 ComponentC 内部使用 A 和 B
var ComponentC = { 
  components: {
    // 自定义组件名： 组件对象
    'component-a': ComponentA,
    'component-b': ComponentB
  }
 }

 new Vue({
  el: '#app',
  components: {
    'component-c': ComponentC
  }
})
```

## 模块系统
### 局部注册
```js
// ComponentC.vue
import ComponentA from './ComponentA'
import ComponentB from './ComponentB'

export default {
  components: {
    ComponentA,
    ComponentB
  },
  // ...
}
```

## `props`
1. 字符串数组
2. 对象形式: 指定 类型 和 默认值 

### 传递静态/动态 Prop
#### 数字 / 数组 / 对象
```html
<!-- 用 `v-bind` 来告诉 Vue 这是一个 JavaScript 表达式而不是一个字符串-->
<blog-post v-bind:likes="42"></blog-post>
<blog-post v-bind:comment-ids="[234, 266, 273]"></blog-post>
<blog-post
  v-bind:author="{
    name: 'Veronica',
    company: 'Veridian Dynamics'
  }"
></blog-post>
```

#### 布尔值
```html
<!-- 包含该 prop 没有值的情况在内，都意味着 `true`-->
<blog-post is-published></blog-post>

<!-- 用 `v-bind` 来告诉 Vue 这是一个 JavaScript 表达式而不是一个字符串-->
<blog-post v-bind:is-published="false"></blog-post>
```

#### 传入一个对象的所有 property
- 使用不带参数的 `v-bind=`
  ```js
  post: {
    id: 1,
    title: 'My Journey with Vue'
  }
  ``` 

  ```html
  <blog-post v-bind="post"></blog-post>

  <!-- 等价于 -->
  <blog-post
    :id="post.id"
    :title="post.title"
  ></blog-post>
  ```

### 验证
- 会在一个组件实例创建之前进行验证
- ∴ 实例的 property `data` `computed` 等在 `default` 或 `validator` 函数中是不可用的

```js
Vue.component('my-component', {
  props: {
    // 基础的类型检查 (`null` 和 `undefined` 会通过任何类型验证)
    propA: Number,
    // 多个可能的类型
    propB: [String, Number],
    propC: {
      type: String,
      // 必填的字符串
      required: true
      // 带有默认值的数字
      default: "abc"

    },
    propD: {
      type: Object,
      // 对象或数组默认值必须从一个工厂函数获取
      default: function () {
        return { message: 'hello' }
      }
    },
    // 自定义验证函数
    propE: {
      validator: function (value) {
        // 这个值必须匹配下列字符串中的一个
        return ['success', 'warning', 'danger'].indexOf(value) !== -1
      }
    },
    propF: {
      // 验证 `author` prop 的值是否是通过 `new Person` 创建的。
      author: Person
    }
  }
})
```


### 禁用Attribute继承
- 除`class` 和 `style`外，其他外部提供的attribute 会替换掉模版里设置好的值
- 组件可以接受任意的 attribute. 如果不希望组件的根元素继承 attribute可以在组件的选项中设置：`inheritAttrs: false`
  - 配合实例的 `$attrs` property 使用: 包括组件 `props` 和 `emits` property 中未包含的所有属性的 attribute名 和 attribute值
  - 具有多个根节点的组件需要显式绑定 `$attrs`: 提示绑定到哪个上面

  ```js
  app.component('custom-layout', {
    template: `
      <header>...</header>
      <main v-bind="$attrs">...</main>
      <footer>...</footer>
    `
  })
  ```

## 自定义事件
- 事件名**不存在**任何自动化的大小写转换：不会被用作一个 JavaScript 变量名或属性名
-  HTML 是大小写不敏感的，所以 `v-on:myEvent` 将会变成 `v-on:myevent` → 导致 myEvent 不可能被监听到

### 自定义组件的`v-model`： `model`
- 组件上的`v-model`会默认绑定 `value` prop 和 `input`事件, 如复选框等输入控件可利用`model`

```js
Vue.component('base-checkbox', {
  model: {
    prop: 'checked',
    event: 'change'
  },
  props: {
    checked: Boolean
  },
  template: `
    <input
      type="checkbox"
      :checked="checked"
      @change="$emit('change', $event.target.checked)"
    >
  `
})
```

```html
<base-checkbox v-model="isSelected"></base-checkbox>
```

1. `isSelected`的值会传入`checked` prop
2. 当组件触发`change`事件，附带一个新的值时，`isSelected`的 property 也会更新 
> 仍然需要在组件的 props 选项里声明 checked 这个 prop。

### 原生事件绑定到组件： `$listeners` property
- `$listeners`包含了 作用在这个组件上的所有监听器，配合 `v-on="$listeners"` 可以指定给组件的特定的子元素
  - 比如根元素不是 `input` 而是`lable` 
  - 配合 `computed` 计算属性 处理这些监听器

### `sync`修饰符 ？？



# Mixin
- 通过`mixins: []` 分发 Vue 组件中的可复用功能

## 选项合并
- 以组件数据优先
- mixin的钩子将在组件自身钩子之前调用

```js
var mixin = {
  data: function () {
    return {
      message: 'hello',
      foo: 'abc'
    }
  }
}

new Vue({
  mixins: [mixin],
  data: function () {
    return {
      message: 'goodbye',
      bar: 'def'
    }
  },
  created: function () {
    console.log(this.$data)
    // => { message: "goodbye", foo: "abc", bar: "def" }
  }
})
```

## 全局mixin
- 会影响每个单独创建的 Vue 实例: `Vue.mixin({...})`

## 自定义指令 `directive`
### 全局自定义指令 
```js
// 创建全局 v-focus
Vue.directive('focus', {
  // 当被绑定的元素插入到 DOM 中时
  inserted: function(el) {
    el.focus();
  }
})
```

### 局部自定义指令
```js
directive: {
  focus: {
    inserted: function(el) {
      el.focus();
    }
  } 
}
```

### 钩子函数
1. `bind`：只调用一次，指令第一次绑定到元素时调用。在这里可以进行一次性的初始化设置。
2. `inserted`：被绑定元素插入父节点时调用 （仅保证父节点存在，但不一定已被插入文档中）。
3.  `update`：所在组件的 VNode 更新时调用，但是可能发生在其子 VNode 更新之前。
4.  `componentUpdated`：指令所在组件的 VNode 及其子 VNode 全部更新后调用。
5.  `unbind`：只调用一次，指令与元素解绑时调用。


### 钩子函数参数
- `el`：指令所绑定的元素，可以用来直接操作 DOM。
- `binding`：一个对象，包含以下 property：
  1. `name`：指令名，不包括 v- 前缀。
  2. `value`：指令的绑定值，例如：`v-my-directive="1 + 1"` 中，绑定值为 2。
  3. `oldValue`：指令绑定的前一个值，仅在 `update` 和 `componentUpdated` 钩子中可用。无论值是否改变都可用。
  4. `expression`：字符串形式的指令表达式。例如 `v-my-directive="1 + 1"` 中，表达式为 `"1 + 1"`。
  5. `arg`：传给指令的参数，可选。例如 `v-my-directive:foo` 中，参数为 `"foo"`。
  6. `modifiers`：一个包含修饰符的对象。例如：`v-my-directive.foo.bar`中，修饰符对象为 `{ foo: true, bar: true }`。
- `vnode`：Vue 编译生成的虚拟节点。移步 VNode API 来了解更多详情。
- `oldVnode`：上一个虚拟节点，仅在 `update` 和 `componentUpdated` 钩子中可用。

> 除`el`外，其他参数应该都是只读，不能修改

- 简写：bind 和 update 触发相同的行为
```js
Vue.directive('color-swatch', function (el, binding) {
  el.style.backgroundColor = binding.value
})
```

## 渲染函数：`render`
```html
<anchored-heading :level="1">Hello world!</anchored-heading>
```

```js
Vue.component('anchored-heading', {
  render: function (createElement) {
    return createElement(
      'h' + this.level,   // 标签名称
      this.$slots.default // 子节点数组
    )
  },
  props: {
    level: {
      type: Number,
      required: true
    }
  }
})
```

## 过渡效果
### 过渡的类名
1. `v-enter`：进入过渡的**开始状态**。在元素被插入之前生效，在元素被插入之后的下一帧移除。
2. `v-enter-active`：定义进入过渡**生效时的状态**, 在整个进入过渡的阶段中应用。在元素被插入之前生效，在过渡/动画完成之后移除。用来定义进入过渡的 过程时间，延迟和曲线函数。
3. `v-enter-to`：定义进入过渡的**结束状态**。在元素被插入之后下一帧生效 (v-enter 同时被移除)，在过渡/动画完成之后移除。
4. `v-leave`：定义离开过渡的开始状态。在离开过渡被触发时立刻生效，下一帧被移除。
5. `v-leave-active`：定义离开过渡生效时的状态。在整个离开过渡的阶段中应用，在离开过渡被触发时立刻生效，在过渡/动画完成之后移除。这个类可以被用来定义离开过渡的过程时间，延迟和曲线函数。
6. `v-leave-to`：定义离开过渡的结束状态。在离开过渡被触发之后下一帧生效 (v-leave 同时被删除)，在过渡/动画完成之后移除。

- 使用了 `<transition name="my-transition">`，那么 `v-` 会替换为 `my-transition-`: 例如 `my-transition-enter`
- `v-enter-active` 和 `v-leave-active` 可以控制进入/离开过渡的不同的缓和曲线

### css过渡
```html
<div id="toggle-example">
  <button @click="show = !show"> Toggle render</button>
  <transition name="slide-fade">
    <p v-if="show">Now you see me</p>
  </transition>
</div>
```

```js
new Vue({
  el: "#toggle-example",
  data(): {
    return {
      show: true
    }
  }
})
```

- 前缀更改为：`.slide-fade-`

```css
.slide-fade-enter-active {
  transition: all .3s ease
}
.slide-fade-leave-active {
  transition: all .8s cubic-bezier(1.0, 0.5, 0.8, 1.0);
}
.slide-fade-enter, .slide-fade-leave-to {
  /* 右移10px消失 */
  transform: translateX(10px);
  opacity: 0;
}
```

### css动画
- 用法和css过渡相同，`v-enter` 类名在节点插入 DOM 后不会立即删除，而是在 `animationend` 事件触发时删除。

```css
.bounce-enter-active {
  animation: bounce-in .5s;
}
.bounce-leave-active {
  animation: bounce-in .5s reverse;
}
@keyframes bounce-in {
  0% {
    transform: scale(0);
  }
  50% {
    transform: scale(1.5);
  }
  100% {
    transform: scale(1);
  }
}
```

### 自定义过渡的类名
- 优先级高于普通的类名，适用于第三方css库
  - `enter-class`
  - `enter-active-class`
  - `enter-to-class `
  - `leave-class`
  - `leave-active-class`
  - `leave-to-class `

### 多个元素的过渡模式
- 当两个按钮过渡中，都被重绘了， 进入和离开同时发生

- `in-out`：新元素先进行过渡，完成之后当前元素过渡离开。
- `out-in`：【大部分】当前元素先进行过渡，完成之后新元素过渡进入。

# 过滤器 `filters`
- 用来过滤数据，不会修改数据，而是过滤数据，改变用户看到的输出
- 放在操作符 `|` 后面进行指示

```js
<li>商品价格：{ {item.price | filterPrice} }</li>
 
 filters: {
    filterPrice (price) {
      return price ? ('￥' + price) : '--'
    }
  }
```

# 常见问题
## v-if、v-show、v-html 的原理
- v-if会调用addIfCondition方法，生成vnode的时候会**忽略对应节点**，render的时候就不会渲染；
- v-show会生成vnode，render的时候也会渲染成真实节点，只是在render过程中会在节点的属性中修改show属性值，也就是常说的`display`；
- v-html会先移除节点下的所有节点，调用html方法，通过addProp添加innerHTML属性，归根结底还是设置`innerHTML`为v-html的值。

## v-if和v-show的区别
- 手段：v-if是动态的向DOM树内添加或者删除DOM元素；v-show是通过设置DOM元素的display样式属性控制显隐；
- 编译过程：v-if切换有一个局部编译/卸载的过程，切换过程中合适地销毁和重建内部的事件监听和子组件；v-show只是简单的基于css切换；
- 编译条件：v-if是惰性的，如果初始条件为假，则什么也不做；只有在条件第一次变为真时才开始局部编译; v-show是在任何条件下，无论首次条件是否为真，都被编译，然后被缓存，而且DOM元素保留；
- 性能消耗：v-if有更高的切换消耗；v-show有更高的初始渲染消耗；
- 使用场景：v-if适合运营条件不大可能改变；v-show适合频繁切换。

## 双向绑定
数据变化 -> 视图更新；
视图交互变化(input) -> 数据 model 变更

## VDom
- 组件内部进行Virtual Dom Diff获取更加具体的差异
- 组件级别运用双向绑定

## 对keep-alive的理解
- 保存一些组件的状态防止多次渲染

### 步骤
- 通过当前组件名去匹配原来 include 和 exclude，判断当前组件是否需要缓存，不需要缓存，直接返回当前组件的实例vNode
- 需要缓存，判断他当前是否在缓存数组里面：
  - 存在，则将他原来位置上的 key 给移除，同时将这个组件的 key 放到数组最后面（LRU）
  - 不存在，将组件 key 放入数组，然后判断当前 key数组是否超过 max 所设置的范围，超过，那么削减未使用时间最长的一个组件的 key

## 子组件不可以直接改变父组件的数据
- 维护父子组件的单向数据流。每次父级组件发生更新时，子组件中所有的 prop 都将会刷新为最新的值
- 只能通过 `$emit` 派发一个自定义事件，父组件接收到后，由父组件修改。

## 生命周期
【beforeCreate】：不能访问到data、computed、watch、methods上的方法和数据。
【created】 ：实例上配置的 options 包括 data、computed、watch、methods 等都配置完成. 但是还未挂载到 DOM，所以不能访问到 $el 属性。
【beforeMount】：在挂载开始之前被调用，相关的render函数首次被调用。此时还没有挂载html到页面上。
【mounted】：在el被新创建的 vm.$el 替换。此过程中进行ajax交互。
【beforeUpdate】：虽然响应式数据更新了，但是对应的真实 DOM 还没有被渲染
【updated】 ：组件 DOM已经更新，所以可以执行依赖于DOM的操作。然而在大多数情况下，应该避免在此期间更改状态
【beforeDestroy】：实例销毁之前调用。这一步，实例仍然完全可用，this 仍能获取到实例。
【destroyed】：实例销毁后调用，调用后，Vue 实例指示的所有东西都会解绑定，所有的事件监听器会被移除，所有的子实例也会被销毁。该钩子在服务端渲染期间不被调用。

### 父组件和子组件生命周期钩子执行顺序
1. 首次加载渲染过程： 父beforeCreate->父created->父beforeMount->子beforeCreate->子created->子beforeMount->子mounted->父mounted
2. 子组件更新过程： 父beforeUpdate->子beforeUpdate->子updated->父updated
3. 父组件更新过程： 父beforeUpdate->父updated
4. 销毁过程： 父beforeDestroy->子beforeDestroy->子destroyed->父destroyed

## 组件通信
- 父组件通过props向子组件传递数据，子组件通过$emit和父组件通信
- 给子组件命名 ref：`<child ref="child"></component-a>`，父组件调用
- `this.$parent.val` 得到父组件的
- `attrs` :继承所有的父组件属性 / `$listeners`:里面包含了作用在这个组件上的所有监听器

## 懒加载如何实现
1. 使用箭头函数+import动态加载： `const List = () => import('@/components/list.vue')`
2. 箭头函数+require动态加载: `component: resolve => require(['@/components/list'], resolve)`