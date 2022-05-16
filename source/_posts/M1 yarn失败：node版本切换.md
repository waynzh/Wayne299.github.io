---
title: 【问题】Mac M1 yarn失败
date: 2021-12-23 14:26:42
categories: FE
tags: [总结, git]
---

# node版本切换
## 解决方案
```bash
arch -x86_64 zsh
source $(brew --prefix nvm)/nvm.sh
nvm use v12

// 查看现在node版本
nvm current node -v
```