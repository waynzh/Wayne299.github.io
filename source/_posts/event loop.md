---
title: 'Event Loop（事件循环）'
date: 2021-10-07 16:45:05
categories: FE
tags: [js基础, 事件循环]
---

# 浏览器 进程 / 线程
- 进程：一个网页就是一个进程
- 线程：
  1. 类别A：GUI 渲染线程 （与主线程互斥）
  2. 类别B：JS 引擎线程 （**主线程**）
  3. 类别C：EventLoop轮询处理线程
  4. 类别D：其他线程，有 定时器触发线程 (setTimeout)、http 异步线程、浏览器事件线程 (onclick)等等。

## 线程
<!--more-->
- **主线程**：执行时会生成 **执行栈**
- 类别D的**异步线程**：等主线程碰到异步代码，就把它丢给各自相对应的线程去执行 + 保存回调函数，等EventLoop轮询处理线程过来取相应的回调函数
- **消息队列**（任务队列）： 一个静态的队列存储结构，非线程，只做存储。存的是一堆**异步成功后**的**回调函数**
- **EventLoop轮询处理线程**

> 如果 主线程 里执行的代码复杂需要很长时间，这时队列里的函数们就排着，等着主线程啥时执行完，再执行消息队列
> 对于setTimeout，setInterval的定时，可能因为主线程里的代码执行很久而不准时
> TODO：如何解决定时误差

## 消息队列：宏任务 & 微任务
1. 宏任务：
  - setTimeout，setInterval，setImmediate，requestAnimationFrame
  - UI交互事件
  - 网络请求等等
2. 微任务：
   - Promise.then
   - process.nextTick(Node.js 环境)
   - Object.observe
   - MutationObserver

- 整体顺序：
  1. 首先，整体的script(作为第一个宏任务)开始执行的时候，会把所有代码分为同步任务、异步任务两部分
  2. 同步任务会直接进入主线程依次执行
  3. 异步任务会再分为宏任务和微任务
  4. 当主线程内的任务执行完毕，主线程为空时，会检查微任务的Event Queue，如果有任务，就全部执行，如果没有就执行下一个宏任务...
- 执行顺序：执行同步代码 → 检查微任务并执行 → GUI渲染 → 执行宏任务1 → 检查微任务并执行 → GUI渲染 → 执行宏任务2 ...
> 每次执行完一个宏任务后都要去检查微任务并全部执行，再进行渲染。

## async/await
- await 以前的代码，相当于与 new Promise 的同步代码
- await 以后的代码相当于 Promise.then的异步


# Node
- 宏任务分好几种类型，分别有不同的任务队列，并且又有顺序区别...

> Node会先执行所有类型为 timers 的 MacroTask，然后执行所有的 MicroTask(NextTick例外)
> 进入 poll 阶段，执行几乎所有 MacroTask，然后执行所有的 MicroTask
> 再执行所有类型为 check 的 MacroTask，然后执行所有的 MicroTask
> 再执行所有类型为 close callbacks 的 MacroTask，然后执行所有的 MicroTask
> 至此，完成一个 Tick，回到 timers 阶段
> ……


# 参考
(JS线程、Event Loop、事件循环、任务队列、宏任务)[https://juejin.cn/post/6844903752621637645]
(JS运行机制)[https://juejin.cn/post/6844904050543034376]
(B站视频)[https://www.bilibili.com/video/BV1kf4y1U7Ln?from=search&seid=12771351763522822849&spm_id_from=333.337.0.0]