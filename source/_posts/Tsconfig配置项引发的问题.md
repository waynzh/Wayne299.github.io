---
title: Tsconfig 未配置 allowJs 引发的问题
date: 2022-07-17 22:41:12
categories: FE
tags: [ts, 总结]
---

# 现象
正常的sfc的vue文件提示提示一下奇怪的JSX问题，甚至有解析成React之类的...

![pic1](Tsconfig配置项引发的问题/pic1.png)
![pic2](Tsconfig配置项引发的问题/pic2.png)


# 解决方法
tsconfig文件的配置有问题，需要设置 `"allowJs": true`
