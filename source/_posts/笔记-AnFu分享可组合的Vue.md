---
title: 笔记 - VueUse 作者 Anthony Fu 分享可组合的 Vue
date: 2022-05-21 13:01:12
categories: FE
tags: [笔记, vue]
---

# Ref & Reactive
能用 Ref 就尽量用
- Ref：
	* Pros： 显示调用 + 类型检查
	* Cons：`.value`
- Reactive： 
	* Pros：自动 Unwrap
	* Cons：在类型上和一般对象没有区别 + ES6解构会失去响应性(toRefs) + 使用watch需要用箭头函数包装

## Ref 自动解包
- `watch` 直接接受 Ref 作为监听对象，回调中为解包后的值
	```js
	const counter = ref(0)
	watch(counter, count => {
		console.log(count); // same as `counter.value`
	})
	```
<!-- more -->

- reactive包含ref，直接修改ref，都会变化
```js
	const counter = ref(0)
	const searchParams = ref({
		count: counter // 不解构直接用ref
	})

	counter.value++ // counter: 1  searchParams.count: 1
```
[参考Vue文档](https://staging-cn.vuejs.org/guide/essentials/reactivity-fundamentals.html#ref-unwrapping-in-reactive-objects)

## 接受 Ref 作为函数参数【模式】
```ts
function add(a: Ref<number>, b: Ref<number>) {
	return computed(() => a.value + b.value)
}
```

- 进阶可以同时接受传入值和Ref
```ts
function add(
	a: Ref<number> | number,
	b: Ref<number> | number
) {
	return computed(() => unref(a) + unref(b))
}

const a = ref(1)
const c = add(a, 5) // 可以省一些 `.value` 操作
c.value = 6
```


## MaybeRef【技巧】

```ts
type MaybeRef<T> = Ref<T> | T

function add(
	a: MaybeRef<number>,
	b: MaybeRef<number>
) {
	return computed(() => unref(a) + unref(b)) // unref 之后一定是 T
}
```

- 不需要管是否嵌套，`ref`和`unref`都可以获得想要的值

### 重复使用已有 Ref【技巧】
```ts
const foo = ref(1)  // Ref<1>
const bar = ref(foo) // Ref<1>

foo === bar // true
```

### unref【技巧】
- 可以作为重置ref值时使用。∵ 不需要管是否isRef


## 请求可以写成“同步”形式【技巧】
```ts
const { data } = useFetch(url).json()
// 利用 ?. 等待结果填充
const user_url = computed(() => data.value?.user_url)
```

## 副作用自动清除【模式】
- Vue 原生的 `watch` 和 `computed` API 会在组件销毁时自动解除内部的监听。
- 设计函数时，需要遵循同样的模式
```ts
export function unseEventLister(target: EventTarget, name: string, fn: any) {
	target.addEventListener(name, fn)

	onUnmounted(() => {
		target.removeEventListener(name, fn) // <--
	})
}
```


## 类型安全的 Provide / Inject【核心】
-  inject时会丢掉上下文共享类型。可以使用 `InjectionKey` 实现类型安全检查

```ts
// context.ts
import { InjectionKey } from 'vue'

export interface UserInfo {
	id: number
	name: string
}

export const User: InjectionKey<UserInfo> = Symbol()
```

```ts
// parent.vue
import { User } from './context'
export default {
	setup() {
		provide(User, {
			id: 8,
			name: 'Wayne' // 类型检查
		})
	}
}
```

```ts
// child.vue
import { User } from './context'
export default {
	setup() {
		const user = inject(User) // UserInfo | undefined

		if (user) {
			console.log(user.name); // 类型检查
		}
	}
}
```

## 状态共享【模式】
### SPA方案（不兼容SSR）
```ts
// shared.ts
export const state = reactive({
	id: 1
	name: 'wayne'
})
```

```ts
// A.vue
import { state } from './shared'
state.id += 1
```

```ts
// B.vue
import { state } from './shared'
console.log(state.id); // 2
```

### 兼容 SSR 的状态共享
- 使用`provide` 和 `inject`来共享应用层面的状态
- Vue Router v4也是类似的方式：`createRouter()` 和 `useRouter`

```ts
// context.ts
export const myStateKey: InjectionKey<MyState> = Symbol()

export function createMyState() { // 插件
	const state = {
		/* ... */ 
	}

	return {
		install(app: App) { // 获取vue instance
			app.provide(myStateKey, state)
		}
	}
}

export function useMyState(): MyState { // 工具函数：调用state
	return inject(myStateKey)! // 当确定是global时，表示非 undefined
}
```

```ts
// main.ts
const App = create(App)

app.use(createMyState())
```

```ts
// A.vue

// 在任何组件中使用这个函数来获取状态对象
const state = useMyState()
```

## useVModal【技巧】
- 一个让使用props和emit更加容易的工具。

```ts
export function useVModal(props, name) {
	const emit = getCurrentInstance().emit

	return computed({
		get() {
			return props[name]
		},
		set(v) {
			emit(`update:${name}`, v)
		}
	})
}
```

```ts
export default defineComponent ({
	setup(props) {
		const value = useVModal(props, 'value')

		return {
			value
		}
	}
})
```

```html
<template>
	<input v-model="value" />
</template>
```




















