---
title: 'overflow:hidden'
date: 2019-11-21 21:10:31
category: FE
tags: css
---

# `overflow:hidden` 的作用总结
## 一. 溢出隐藏
- 一般在div中设置后，该元素的内容若超出了给定的宽度和高度属性，那么超出的部分将会被隐藏，不占位。

```css
text-overflow: ellipsis;	/* 当对象内文本溢出时可显示省略标记（...）*/
```

## 二. 消除浮动
- 一般而言，父级元素不设置高度时，高度由随内容增加自适应高度。当父级元素内部的子元素全部都设置浮动float之后，子元素会脱离标准流，不占位，父级元素检测不到子元素的高度，父级元素高度为0，下面的元素会顶上去，造成页面的塌陷。

### 方法一：给父级加`overflow: hidden`属性

### 方法二：给父级加`::after`

```css
.outDiv::after {
    content: "";
    clear: both;
    display: block;
}
```
<!--more-->

### 方法三：设置`width：xxpx`
- 这样父级的高度就随子级容器及子级内容的高度而自适应。

## 三. 解决外边距塌陷
- 父级元素内部有子元素，如果给子元素添加`margin-top`样式，那么父级元素也会跟着下来，造成外边距塌陷。
- 给父级元素添加`overflow:hidden`属性，父级元素保持不变，而只会对子元素有影响。
- 当使用`float: right`时，不需要考虑塌陷（在设置父级元素width/height的前提下）。