---
title: Event Bus
date: 2021-8-25 14:48:22
categories: FE
tags: [vue]
---

# 初始化
- EventBus 是一个不具备 DOM 的组件，仅仅具有它的实例方法
- 基座中使用的是全局注册：即 发布/订阅模式
```js
// event-bus.js
import Vue from 'vue'
export const EventBus = new Vue()

// 全局注册 不需要注册
Vue.prototype.$eventBus = new Vue()
// 或者
import { EventBus } from './eventBus';
Vue.prototype.$eventBus = new EventBus()
```
<!--more-->

# 发布事件 / 订阅事件
```js
// 发布消息
this.$eventBus.$emit(channel: string, callback(payload1,…))

// 监听订阅消息
this.$eventBus.$on(channel: string, callback(payload1,…))
```

# 移除事件监听
- A向eventBus发送事件，重新进入B会创建多次监听，所以要及时销毁
```js
this.$eventBus.$off('aMsg')
```