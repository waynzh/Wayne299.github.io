---
title: Intro to Git
date: 2019-8-20
---
# Git 语法入门

## Terminal

- <kbd>command</kbd> + K ： clear
- mv ：改名 / 移动文件到文件夹

## Git
### Add All

```bash
git add .
git add A
```
<!--more-->

### 取消、修改、恢复
- **取消已经暂存的文件**。即，撤销先前"git add"的操作

```bash
git rm --cached <file>...
```

- **取消对文件的修改**。还原到最近的版本，废弃本地做的修改。

```bash
git checkout -- <file>
```

- **修改最后一次提交**。用于修改上一次的提交信息，或漏提交文件等情况。

```bash
git commit --amend
```

### 获取log

```bash
git log --oneline 
git log --oneline --all --graph --decorate
```

### Branch

```bash
git branch				// 列出有的branch
git branch -d <branch_name>		// 删除分支 (⚠️一般没必要)
git checkout -b <branch_name>		// 新建分支 --> 自动切换到新分支
git checkout <branch_name>	  	// 切换branch
(master) git merge <branch_name>          // 将分支merge进master中
(master) git merge --squash <branch_name> // combine为新的分支
```

- git log --> <kbd>enter</kbd> --><kbd>Q</kbd> --> git checkout <hashcode> -->新建branch进行更新(update file)


## Github

```bash
git init
git add README.md
git commit -m "add README.md"
git remote add origin git@github.com:Wayne299/xxx.git
git push -u origin master
```
