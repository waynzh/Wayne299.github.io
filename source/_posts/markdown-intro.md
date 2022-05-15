---
title: MarkDown Syntax Intro
date: 2019-7-30
categories: Tools
tags: markdown
---
# Markdown 语法入门
## 字体

- 要加粗的文字左右分别用两个\*号包起来  
- 要倾斜的文字左右分别用一个\*号包起来  
- 要倾斜和加粗的文字左右分别用三个\*号包起来  
- 要加删除线的文字左右分别用两个\~号包起来 
 
显示结果如下: 
<!--more-->
- **加粗**  
- *斜体*  
- ***斜体加粗***  
- ~~删除线~~  


Option name         | Markup           |  Example    |
--------------------|------------------|:-----------:|
Intra-word emphasis | So A\*maz\*ing   | So A*maz*ing |
Strikethrough       | \~\~Much wow\~\~ | ~~Much wow~~ |
Underline           | \_So doge\_      | _So doge_    |
Quote               | \"Such editor\"  | "Such editor"|
Highlight           | \==So good\==    | ==So good==  |




## 列表

- 文字前加上 \- \+ \* 任何一种都可以变为无序列表，加上 1. 2. 3. 等编号则变为有序列表。
- 列表嵌套，上一级和下一级之间敲两个空格即可。

显示结果如下：  

* 无序1
  1. 有序1
  2. 有序2
  3. 有序3
* 无序2
  1. 有序1
  2. 有序2
  3. 有序3

## 引用

- 引用超链接，\[链接文本](链接 URL 地址),如: \[Google](https://www.google.com)
- 或者 <链接地址>, 如: \<https://www.google.com>
- 或者在文档的**结尾**为变量赋值（网址）

显示结果如下：

- [Google](https://www.google.com) 
- <https://www.google.com> 
	
## 图片

- 添加图片语法：\!\[alt 属性文本](图片地址)  \!\[alt 属性文本](图片地址 "title属性的标题") 

显示结果如下：  
![图标](https://www.google.com/url?sa=i&source=images&cd=&ved=2ahUKEwif3Yy11NzjAhXaR30KHb4oCf4QjRx6BAgBEAU&url=https%3A%2F%2Fwww.blog.google%2F&psig=AOvVaw016ik8oz3SNvau2l54VeLe&ust=1564576665173064)
	
## 代码

- 用\` \`包裹行内内容
- 用 \`\`\` 包裹多行

显示结果如下：

这是一段`代码`

```javascript
$(document).ready(function () {
    alert('Wayne');
});
```

## 分割线

- 三个以上的星号(*)、减号(-)、底线(_)即，但可行内不能有其他东西

显示结果如下：  

********* 
-----
_____

## 表格

- “-” 有一个就行，为了美观对齐，多加了几个
- 文字默认居左 ，“-”两边加：表示文字居中，“-”右边加：表示文字居右。

显示结果如下：  

表头|表头|表头
---|:--:|---:
内容|内容|内容
内容|内容|内容

## html元素

- 包括\<kbd\> \<b\> \<i\> \<em\> \<sup\> \<sub\> \<br\>等 


## 剩余

- 流程图 / 数学公式 等