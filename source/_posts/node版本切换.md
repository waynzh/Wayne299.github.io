---
title: 【问题】node版本切换
date: 2021-12-23 14:26:42
categories: FE
tags: [总结, git]
---

# nvm 切换版本
## 解决方案
```bash
arch -x86_64 zsh
source $(brew --prefix nvm)/nvm.sh
nvm use v12

// 查看现在node版本
nvm current node -v
```

# 终端每次重新打开都找不到nvm
## 解决方案
```bash
# 因为使用了oh-my-zsh，所以需要将以下配置添加到.zshrc文件中
source $(brew --prefix nvm)/nvm.sh
```

# nvm 切换默认版本
## 解决方案
```bash
nvm ls
nvm alias default v14.18.3
```
