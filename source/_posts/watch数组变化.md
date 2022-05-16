---
title: 【问题】watch 数组变化
date: 2022-4-20 21:38:22
categories: FE
tags: [总结, vue]
---

# watch ref的值 / 地址
## 问题现象
- watch 的 ref 值不加 `.value` 只是地址，加 `.value` 可以watch到值的变化。

## 解决方案
```js
const timeRange = ref([])
watch(() => timeRange.value, () => {
  [searchParams.createStartTime, searchParams.createEndTime] = timeRange.value
  resetPageSetting()
  debounceChange()
})
```