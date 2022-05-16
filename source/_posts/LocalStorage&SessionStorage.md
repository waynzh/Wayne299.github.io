---
title: 'LocalStorage & SessionStorage 比较'
date: 2021-10-25 21:45:42
categories: FE
tags: [js基础，总结]
---

# LocalStorage & SessionStorage
- 刷新当前页面，或者通过`location.href`、`window.open`、或者通过带`target="_blank"`的a标签打开新标签，会把旧窗口（或标签）的sessionStorage数据带过去.
  - 但从此之后，新窗口（或标签）的sessionStorage的增删改和旧窗口已经没有关系了。
  - 如果只是在当前标签内跳转新页面（或者刷新），数据还会保留（前提当然是同域）。
- 但是如果是主动打开一个新窗口或者新标签，那就没有sessionStorage。

∴ sessionStorage的session仅限**当前标签页**或者**当前标签页打开的新标签页**。通过其它方式新开的窗口或标签不认为是同一个session




