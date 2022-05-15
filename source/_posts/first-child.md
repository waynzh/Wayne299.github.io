---
title: first-child易错问题
date: 2019-11-30 18:42:19
category: FE
tags: css
---

# first-child等伪类选择器易错问题

## E:first-child {}
- 匹配父元素的第一个子元素E。
- 要使该属性生效，E元素必须是某个元素的第一个**子元素**，且是**E属性**。


```html
<div>
    <h1>h1</h1>
    <p>p1</p>
    <p>p2</p>
    <p>p3</p>
</div>
```
- 若在这时选择第一个p元素，使用`p:first-child`则会出现错误，因为p的父元素是**div**，而对于div来说，它的**第一个子元素是h1**，不是p，所以会出错。
- 同理，`E:last-child {}` `E:only-child {}` 与上面的一样，E元素必须是其父元素的最后一个子元素或唯一一个子元素才可以。

## E:nth-child(1) {}
- 与`E:first-child {}`等效，E需要是父元素的第一个子元素。
- 比如： `p:nth-child(2)` 才可以选中p1

## :nth-child(odd/even) {}
- 匹配属于其父元素的第N个子元素，不论元素的类型。

## 规范
- 可写成 `div h1:first-child{}` 明确父元素，自然能看到是不是第一个E属性子元素，就不容易犯错。