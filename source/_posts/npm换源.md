---
title: 【问题】npm换源
date: 2022-5-3 18:00:22
categories: FE
tags: [总结]
---

# 查看当前源
```bash
npm config get registry
```

# 切换源
1. 官方源
```bash
npm config set registry https://registry.npmjs.org/
```

<!--more-->

2. 公司内部
```bash
npm config set registry http://r.npm.sankuai.com
```

