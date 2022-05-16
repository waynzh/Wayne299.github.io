---
title: path.join & path.resolve
date: 2021-8-24 14:06:12
categories: FE
tags: [总结, js基础]
---

# path.join 和 path.resolve区别
## `path.join([path1][, path2][, ...])` 连接路径
```js
var path = require('path'); 
//合法的字符串连接 
path.join('/foo', 'bar', 'baz/asdf', 'quux', '..') 
// 连接后  '/foo/bar/baz/asdf' 
```

## `path.resolve([from ...], to)` 路径解析
- 解析为一个规范化的绝对路径
- 类似于对这些路径逐一进行 cd 操作
<!--more-->

```js
path.resolve('/foo/bar', './baz') 
// 输出结果为  '/foo/bar/baz' 
path.resolve('/foo/bar', '/tmp/file/') 
// 输出结果为  '/tmp/file' 

path.resolve('wwwroot', 'static_files/png/', '../gif/image.gif') 
// 当前的工作路径是 /wayne，则输出结果为 '/wayne/wwwroot/static_files/gif/image.gif'
```