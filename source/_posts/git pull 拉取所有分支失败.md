---
title: 【问题】git pull 失败
date: 2021-8-17 11:54:22
categories: FE
tags: [总结, git]
---

# git pull 拉取所有分支失败
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