---
title: 【问题】本地起服务时 Error
date: 2022-5-05 15:50:21
categories: FE
tags: [总结]
---

# Error: listen EADDRNOTAVAIL
## 问题现象
![EADDRNOTAVAIL](EADDRNOTAVAIL/EADDRNOTAVAIL.png)

## 解决方案
- 原因：SwitchHosts设置的IP地址出错

1. 查看本机ip地址

```shell
ifconfig | grep "inet " | grep -v 127.0.0.1
```

2. 更改SwitchHosts的localhost ip设置