---
title: Intro to Git
date: 2019-8-20
categories: FE
tags: git
---

# Git 语法入门
## 常用Git指令
### Add All

```bash
git add .
git add A
```
<!--more-->

### 取消、修改、恢复
1. ※获取上一条命令的撤销指令※: `gitjk` 
2. **取消已经暂存的文件**。即，撤销先前"git add"的操作 : 
   `git rm --cached <file>...`
3. **取消对文件的修改**。还原到最近的版本，废弃本地做的修改:
	`git checkout -- <file>`
4. **修改最后一次提交**。用于修改上一次的提交信息，或漏提交文件等情况。
   `git commit --amend`
   - 用`rebase -i`把提交挪到最前端就可以修改
   - 或者直接`cherry-pick`

- 本地分支中**回退**几个提交记录来实现撤销改动: 变更还在，但是处于未加入暂存区状态
   `git reset HEAD~1`
- 远程分支中**撤销提交**C2后面多了一个新提交C2‘，C2‘ = C1
	`git revert HEAD`

### 获取log

```bash
git log --oneline 
git log --oneline --all --graph --decorate
```

### Branch
#### 基本操作
```bash
git branch				// 列出有的branch
git branch -d <branch_name>		// 删除分支 (⚠️一般没必要)
git checkout -b <branch_name>		// 新建分支 --> 自动切换到新分支
git checkout <branch_name>	  	// 切换branch
(master) git merge <branch_name>          // 将分支merge进master中
(master) git merge --squash <branch_name> // combine为新的分支
```

- git log --> <kbd>enter</kbd> --><kbd>Q</kbd> --> git checkout <hashcode> -->新建branch进行更新(update file)

#### 分支合并
1. `git rebase main bugFix`: 在bugFix分支移到main分支上 → 之后切到main再更新main分支`git rebase bugFix`
2. `git cherry-pick bugFix`

#### 相对定位
- 使用 `^` 向上移动 1 个提交记录，如`main^`相当于main的父节点
  - `HEAD^2` 表示第二个父节点，可以帮助合并后的移动
  - 操作符支持链式操作 `git checkout HEAD~^2~2`
- 使用 `~<num>` 向上移动多个提交记录，如 `~3`

```bash
git checkout <SHA-1>^ 
git checkout HEAD^ 
git checkout <branch_name>^
```

#### `-f`: 让分支指向另一个提交
`git branch -f main HEAD~3`: 将 `main` 分支强制指向 HEAD 的第 3 级父提交


### tag
#### git tag
- 永远指向某个提交记录的标识 （当然标签可以被删除后重新在另外一个位置创建同名的标签）: 给指定的提交记录（默认是HEAD）加标签v1
  `git tag v1 <SHA-1>` 
- 可以直接`git checkout v1`

#### git describe
1. 当查询的提交记录上有某个标签，只输出 `<tagName>`
2. `<tag>_<numCommits>_g<hash>`

### remote
#### fetch
1. 从远程仓库下载本地仓库中缺失的**提交记录**
2. 更新远程分支指针(如 origin/main)
   - 不会改变本地仓库的状态：即 不会更新你的 main 分支 + 不会修改你磁盘上的文件

#### pull
`git pull` = `git fetch` + `git merge`

#### 历史偏移
1. 将要上传的工作 移动到远程仓库最新的提交记录下：
   步骤：`git fetch` → `git rebase origin/main` → `git push`
   - 简写：`git pull --rebase` = `git fetch` + `git rebase`
2. 合并新变更
   步骤：`git fetch` → `git merge origin/main` → `git push`
   - 简写：`git pull` → `git push`
   
#### 远程服务器拒绝
先reset到和远程仓库一致 → 新建一个分支feature, 推送到远程服务器`git push origin feature`

#### 远程跟踪
1. 新建分支，让其跟踪远程仓库中的 main：
  `git checkout -b newBranch origin/main` → `git pull`/`git push`
2. 远程追踪分支:
   `git branch -u origin/main newBranch` 在当前分支可省略最后一个参数

#### push origin <source>:<destination>
- `git push origin <source>` 或者 更细致的`<source>:<destination>` 不需要HEAD的指向？？？

## 其他指令
- <kbd>command</kbd> + K ： clear
- mv ：改名 / 移动文件到文件夹
- 修改b.txt: `echo "context" > b.txt`
- 查看git仓库：`cd .git` `cd obkects` `ls -la`
- 查看文件内容：`git cat-file -p <SHA-1>`
- 查看index缓存区： `git ls-file --stage`
- 查看HEAD文件：`cat HEAD`  // 只有commit后才有文件

## Github
- 创建

```bash
git init
git add README.md
git commit -m "add README.md"
git remote add origin git@github.com:Wayne299/xxx.git
git push -u origin master
```
 
- 远程仓库：`pull` 

- 暂存： `stash`  查询：`stash list` 调出：`stash apply`

# 方糖课：版本管理和Git入门
>> 版本管理和Git的设计
>> Git的常用命令
>> Git相关工具

## 版本管理功能
- **全量存储方案**（代码/文本） VS    增量方案（视频/大型二进制）
- 存储在版本库：`.git`目录  ⚠️忽略掉不上传git，有之前的代码信息

### 文件命名
- SHA-1值：文件内容作为文件命名

### 树结构和暂存区 (也是树结构)
code         | index暂存区          |  SHA-1    | | 
--------| ----------------|--------|:-----------:|
在    	   |          在 			 |  不匹配 |   	modified | 
不存在     |          在			 | 		| removed | 
在		   |   不存在              | 		|  added | 



index暂存区    | .git         |  SHA-1    |     | 
-----------|--------|---------|:-----------:|
在    	   |          在 	     | 不匹配     |  	modified |
不存在     |          在    	 | 		    | missing | 
在		   |      不存在       | 		|  untracked | 

### 快照链表commit
- `HEAD`指针移动指向不同快照 （或者checkout SHA-1值）

### 协同(特殊网络情况)
- 本地仓库 和 远程仓库 整合

### 冲突
- 后提交的人解决冲突 （早提交的人早下班

### 分支
- `git checkout -b <branch>` 分支后各开各的功能

### 合并
1. `Merge`: 两个feature和共同祖先**三方合并**，形成新的提交。
2. `Rebase`: 找到共同提交，撤销某一分支的所有permit，先保存到临时目录下；将另一分支的permit应用在这一分支上，再把临时保存的提交应用在之后，形成直线型链表。
	- 会修改远程仓库的原始提交，推荐只在**本地仓库**rebase
	- `git rebase -i HEAD~4` interactive交互式操作
3. `cherry-pick`: 别的分支**特定的**permit合并到本分支上。

# git快捷键 - oh my zsh 
- ga = git add
- gb = git branch
- gc = git commit -m
- gd = git diff
- gf = git fetch
- gs = git status

* gcl = git clone
* gps = git push
* gpl = git pull
* gco = git checkout
* glg = git log
* grb = git rebase


# 参考
 ![git命令速查表](intro-to-git/git.png)
[Learn Git Branching + 沙盘](https://learngitbranching.js.org/?locale=zh_CN)

