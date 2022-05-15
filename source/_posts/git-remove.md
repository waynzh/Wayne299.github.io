---
title: Remove
date: 2019-8-29
categories: FE
tags: git
---
# Git中remove操作

## Push an existing repository from the command line

``` bash
$ cd <file_name>
$ git init
$ git remote add origin git@github.com:Wayne299/x.git
$ git add <file_name>
$ git push -u origin master
```
<!--more-->


##  fatal: 远程 origin 已经存在

``` bash
$ git remote rm origin
$ git remote add origin <SSH_Key>
```

