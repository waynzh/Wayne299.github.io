---
title: props 的 default 是对象和数组
date: 2021-9-1 20:33:02
categories: FE
tags: [总结, vue]
---

# Props & PropType

```js
props: {
  editServiceConfig: {
      type: Object as PropType<ServiceItem>,
      required: false,
      default: () => {}
    }
}
```

## PropType
- 为了**类型推论**

## Props 的 默认值
- 对象或数组的 `default` 必须从一个工厂函数获取
- `props` 会在一个组件实例创建之前进行验证，所以实例的属性 (如 data、computed 等) 在 `default` 或 `validator` 函数中是不可用的。

