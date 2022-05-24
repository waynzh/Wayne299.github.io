---
title: git配置多个ssh key
date: 2022-05-21 11:26:12
categories: FE
tags: [git]
---

# git配置多个ssh key
## 解决方案
1. 同步所有远程分支
```bash
git branch -r | grep -v '\->' | while read remote; do git branch --track "${remote#origin/}" "$remote"; done
```

2. 将本地所有分支与远程保持同步 
```bash
git fetch --all
```

3. 最后拉取所有分支代码 
```bash
git pull --all
```