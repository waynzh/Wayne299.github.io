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












