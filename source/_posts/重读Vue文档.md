---
title: 重读Vue文档
date: 2022-06-11 22:46:12
categories: FE
tags: [笔记, vue, 总结]
---

# Ref & Reactive
## Ref Unwrapping in Reactive Objects
[参考Vue文档](https://vuejs.org/guide/essentials/reactivity-fundamentals.html#ref-unwrapping-in-reactive-objects)
reactive包含ref，直接修改ref，都会变化
```js
	const counter = ref(0)
	const searchParams = ref({
		count: counter // 不解构直接用ref
	})

	counter.value++ // counter: 1  searchParams.count: 1
```

# Conditional Rendering
[参考Vue文档](https://vuejs.org/guide/essentials/conditional.html#v-if-on-template)
## `v-if` on `<template>`
控制包含多个元素时，可以用 `<template>` 包裹（渲染出来时不包含template元素），∴不需要再套一层`div`
```html
<template v-if="ok">
  <h1>Title</h1>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
</template>
```
<!-- more -->

`v-show`不支持写在 `<template>`上

## `v-if` with `v-for`
- vue3：`v-if` > `v-for`
- vue2：`v-if` < `v-for`


# Dynamic Arguments
[参考Vue文档](https://vuejs.org/guide/essentials/template-syntax.html#directives)
##  Value Constraints
`string | null`

##  Syntax Constraints
- html attribute names 不能有空格和引号。∴ 可以用computed来代替直接写在template中。
- 直接使用in-DOM template时（直接写在hmtl文件里的），要避免大写字母。∵ 浏览器会强制转为小写

# List Rendering
[参考Vue文档](https://vuejs.org/guide/essentials/list.html#v-for-with-an-object)
## `v-for` with an Object
顺序依据的是 `Object.keys()`
```html
<li v-for="(value, key, index) in myObject" :key="value">
  {{ index }}. {{ key }}: {{ value }}
</li>
```

- `v-for`支持写在 `<template>`上。
- 若遇到`v-for`和`v-if`一起，推荐在`<template>`上写`v-for`，`v-if`就可以访问到`v-for`scope中的值啦

## Replacing an Array
- 用 `filter()`, `concat()` 和 `slice()`不会改变原数组的方法时：需要用生成的新数组代替旧的
- 在 `computed property`中使用改变原数组的方法时：需要先复制一个再进行操作 `return [...numbers].reverse()`

### 高效
- 不会「丢弃现有的 DOM 并重新渲染整个列表」
- TODO：原因

# Watchers
[参考Vue文档](https://vuejs.org/guide/essentials/watchers.html#basic-example)
## 推荐写法
1. watch single ref: 直接watch
```ts
const value = ref("value")
watch(value, (newVal) => {
  console.log(`value is ${newVal}`)
})
```

2. watch a property of a reactive object: getter写法
```ts
const config = ref({
	name: 'waynzh'
})
watch(() => config.name, (newName) => {
  console.log(`name is ${newName}`)
})
```

## Deep Watchers - getter写法返回 Reactive Object
- 问题：用getter写法返回 Reactive Object时，只有整个替换object才能被监听到。
- 解决方式：
	1. 不用getter写法就自带deep
	2. 显式加 `{ deep: true }`

```ts
const config = ref({
	name: 'waynzh'
})
watch(() => config, (newVal, oldVal) => {
  // `newValue` will be equal to `oldValue`
  // *unless* config 被整个替换
}, { deep: true })
```

- `newValue` 和 `oldValue`是相等的, 除非 Object 被整个替换掉

## `watchEffect`
- 副作用发生期间追踪依赖，但响应性依赖关系不那么明确。勉强可以替代 `watch` 中的 `immediate`

```ts
const url = ref('https://...')
const data = ref(null)

watch(url, async () => {
	const response = await fetch(url.value)
	data.value = await response.json()
}, { immediate: true })

// 收集到能访问到的响应式 property
watchEffect(async () => {
  	const response = await fetch(url.value)
  	data.value = await response.json()
})
```

## Callback Flush Timing - 回调时机
- 默认，watch会在组件更新之前被调用，∴ 在watch中访问的DOM的是更新之前的状态
- 若想访问更新之后的DOM，需要指明`flush: 'post'`
- `watchEffect()` 中，可以使用别名 `watchPostEffect()`

```ts
watch(url, async () => {
	const response = await fetch(url.value)
	data.value = await response.json()
}, { flush: 'post' })
```

# Template Refs
[参考Vue文档](https://vuejs.org/guide/typescript/composition-api.html#typing-component-template-refs)
## 组件ref TS支持： `InstanceType`
- `typeof` 获得组件的类型，再用 `InstanceType` 获取其实例类型
```ts
import MyModal from './MyModal.vue'

const modal = ref<InstanceType<typeof MyModal> | null>(null)

const openModal = () => {
  modal.value?.open()
}
```

## Refs inside `v-for`
- 可以为一个数组，但顺序 *不一定* 是array的顺序

```html
<script setup>
const list = ref([
  /* ... */
])

const itemRefs = ref([]) // 顺序 *不一定* 是array的顺序
</script>

<template>
  <ul>
    <li v-for="item in list" ref="itemRefs">
      {{ item }}
    </li>
  </ul>
</template>
```

## setup 显式暴露子组件内容
- 使用了 `<script setup>` 的组件是默认私有的，除非子组件在其中通过 `defineExpose` 宏显式暴露

```html
<script setup>
import { ref } from 'vue'

const a = 1
const b = ref(2)

// 显式暴露
defineExpose({
  a,
  b
})
</script>
```


# Components Basics
## props TS支持：`defineProps`
[参考Vue文档](https://vuejs.org/guide/typescript/composition-api.html#typing-component-props)
### 基本用法
```html
<script setup lang="ts">
// runtime
const props = defineProps({
  foo: { type: String, required: true },
  bar: Number
})

// type-based
const props = defineProps<{
  foo: string
  bar?: number
}>()
</script>
```

### 限制：不用使用import的interface
```TS
// ✅
interface Props {/* ... */}
defineProps<Props>()

// ❌
import { Props } from './other-file'

// NOT supported
defineProps<Props>()
```

## emit TS支持: `defineEmits`
```html
<script setup lang="ts">
// runtime
const emit = defineEmits(['change', 'update'])

// type-based
const emit = defineEmits<{
  (e: 'change', id: number): void
  (e: 'update', value: string): void
}>()
</script>
```

# Components In-Depth
推荐写法：组件名用 PascalCase，prop用 kebab-case
```html
<MyComponent greeting-message="hello" />
```

## Binding Multiple Properties Using an Object
不加参数直接绑定object，相当于分别绑定：

```html
<script setup lang="ts">
const post = {
  id: 1,
  title: 'My Journey with Vue'
}
</script>

<template>
<!-- 不加参数直接绑定object -->
<BlogPost v-bind="post" />

<!-- 相当于分别绑定 -->
<BlogPost :id="post.id" :title="post.title" />
</template>
```

## Events
[参考Vue文档](https://vuejs.org/guide/components/events.html#usage-with-v-model)
### 最佳实践
组件的双向传值的两种最佳实现：
	1. 父组件 `v-model="value"`，自组件分别绑定值和事件
	2. 子组件 `v-model="value"`绑定值，并利用 `computed` 设置`get()` 和 `set()`计算值

```html
<!-- CustomInput.vue -->
<script setup>
defineProps(['modelValue'])
defineEmits(['update:modelValue'])
</script>

<template>
  <input
    :value="modelValue"
    @input="$emit('update:modelValue', $event.target.value)"
  />
</template>

<!-- index.vue -->
<CustomInput v-model="searchText" />

<!-- 相当于 -->
<CustomInput
  :modelValue="searchText"
  @update:modelValue="newValue => searchText = newValue"
/>
```

```html
<!-- CustomInput.vue -->
<script setup>
const props = defineProps(['modelValue'])
const emit = defineEmits(['update:modelValue'])

const value = computed(() => {
	get() {
		return props.modelValue
	},
	set(val) {
		emit('update:modelValue', val)
	}
})
</script>

<template>
  <input v-model="value" />
</template>
```

### `v-model` arguments
- by default: `modelValue` 为 prop，`update:modelValue` 为emit事件
- 添加一个变量作为指定的prop：

```html
<!-- index.vue -->
<MyComponent v-model:title="bookTitle" />

<!-- MyComponent.vue -->
<script setup>
defineProps(['title'])
defineEmits(['update:title'])
</script>
```

### Handling `v-model` modifiers
- by default: `v-model="value"`， 会有一个`modelModifiers` 的prop
- 添加一个变量后: `v-model:title="value"`，props中modifiers 的规则为`arg+"Modifiers"`

```html
<!-- index.vue -->
<MyComponent v-model.capitalize="text" />

<!-- MyComponent.vue -->
<script setup>
const props = defineProps({
  modelValue: String,
  modelModifiers: { 
  	type: Object,
  	default: () => {}
  }
})

console.log(props.modelModifiers); // { capitalize: true }
</script>
```

```html
<!-- index.vue -->
<MyComponent v-model:title.capitalize="text" />

<!-- MyComponent.vue -->
<script setup>
const props = defineProps({
  title: String,
  titleModifiers: { // `arg + "Modifiers"`
  	type: Object,
  	default: () => {}
  }
})
</script>
```

## Fallthrough Attributes
### In js: `useAttrs()` / `setup(props, {attrs})`
- `const attrs = useAttrs()`
- `setup(props, {attrs}) {}`

### Not reactive
- 如果需要动态变化，可以使用prop 或者 `onUpdated()`

## Slots
[参考Vue文档](https://vuejs.org/guide/components/slots.html#named-slots)

### Named Slots
- 子组件：用name命名slot `<slot name="header">`
- 父组件：用`v-slot:header` 使用

```html
<!-- index.vue -->
<BaseLayout>
  <template v-slot:header>
    <!-- content for the “header” slot -->
  </template>
</BaseLayout>

<!-- BaseLayout.vue -->
<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>
```

![参考图片](https://vuejs.org/assets/named-slots.ebb7b207.png)

### Dynamic Slot Names
语法：`<template v-slot:[dynamicName]>`

### Scoped Slots
子组件内的参数传给父组件，在父组件中访问并安排样式。详情参考 [List生成例子](https://vuejs.org/guide/components/slots.html#fancy-list-example)

- 写法类似 render functions中的逻辑
- 和 Composable 概念类似，Renderless Component即只处理逻辑（fetching，pagination），不处理样式。


## Provide / inject
[参考Vue文档](https://vuejs.org/guide/components/provide-inject.html#inject)
### inject Default Values
- 如果没有匹配到provide的Key（`’message'`），则使用default value
- ∵ 运行时不知道是否会提供，inject的type会默认带有 `undefined`。
	- 如果设置了default value就会移除
	- 如果确认肯定会提供，可以断言 `inject('foo') as string`


```ts
const value = inject('message', 'default value')
```

### 最佳实践 - 将更改保持在`provide`中
- Keep any mutations to reactive state inside of the provider whenever possible
	- provide mutation methods 
	- wrap the provided value with `readonly()`

### 最佳实践 - Provide Symbol Keys
- Provide Symbol Keys
- export the Symbols in a dedicated file


### `InjectionKey`

```ts
import type { InjectionKey } from 'vue'

const key = Symbol() as InjectionKey<string>

provide(key, 'foo') // providing non-string value will result in error
```

# 异步组件
## `defineAsyncComponent`
- 最后得到的 `AsyncComp` 是一个包装器组件，仅在页面需要它渲染时才调用加载函数。

```ts
import { defineAsyncComponent } from 'vue'

const AsyncComp = defineAsyncComponent(() =>
	import('./components/MyComponent.vue')
)

onst AsyncComp = defineAsyncComponent(() => {
	// 加载函数
	loader: () => import('./Foo.vue'),

	// Loading 组件
	loadingComponent: LoadingComponent,
	// 展示加载组件前的延迟时间，默认为 200ms
	// 防止网络很好，看起来像在闪烁
	delay: 200,

	// Error 组件
	errorComponent: ErrorComponent,
	// timeout 时间限制，超时了会显示报错组件，默认值是：Infinity
	timeout: 3000
})
```

# Custom Drectives
[参考Vue文档](https://vuejs.org/guide/reusability/custom-directives.html#introduction)
- setup中，任何`v`开头的都可以作为自定义指令。
- 没有setup中，需要用 `directives` 注册

```html
<script setup>
// enables v-focus in templates
const vFocus = {
  mounted: (el) => el.focus()
}
</script>

<template>
  <input v-focus />
</template>
```

# Build-in  Components
## KeepAlive 
[参考Vue文档](https://vuejs.org/guide/built-ins/keep-alive.html#include-exclude)
### Include / Exclude

```html
<!-- 逗号分隔，不需要v-bind -->
<KeepAlive include="a,b">
  <component :is="view" />
</KeepAlive>

<!-- 正则 -->
<KeepAlive :include="/a|b/">
<!-- Array -->
<KeepAlive :include="['a', 'b']">
```

### Max Cached Instances
超过 max 时，最旧访问的缓存实例将被删除，为新的缓存提供空间

```html
<KeepAlive :max="10">
  <component :is="activeComponent" />
</KeepAlive>
```

### Lifecycle of Cached Instance
- `onDeactivated()`: 
	- `<KeepAlive>`中的cache当从DOM移除时。非 `unmounted`
	- unmount 和 每次从DOM移除时
- `onActivated()`: 
	- `<KeepAlive>`中的cache添加到DOM时。
	- intial mount 和 每次插入DOM时
	


































