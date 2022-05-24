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


