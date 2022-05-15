---
title: Inline-block vs Float
date: 2019-11-5
category: FE
tags: css
---

# inline-block和float的区别
## 文档流（Document flow）
- 浮动元素会脱离文档流，并使得周围元素**环绕这个元素**。而inline-block元素仍在文档流内。因此设置inline-block不需要**清除浮动**。当然，周围元素不会环绕这个元素，你也不可能通过清除inline-block就让一个元素跑到下面去。

## 水平位置（Horizontal position）
- 很明显你不能通过给父元素设置`text-align:center`让浮动元素居中。事实上定位类属性设置到父元素上，均不会影响父元素内浮动的元素。但是父元素内元素如果设置了`display：inline-block`，则对父元素设置一些定位属性会影响到子元素。（这还是因为浮动元素脱离文档流的关系）。
<!--more-->

## 垂直对齐（Vertical alignment）
- inline-block元素沿着默认的基线对齐。浮动元素紧贴顶部。你可以通过vertical属性设置这个默认基线，但对浮动元素这种方法就不行了。这也是我(原作者)倾向于inline-block的主要原因。

## 空白（Whitespace）
- inline-block包含html**空白节点**。如果你的html中一系列元素每个元素之间都换行了，当你对这些元素设置inline-block时，这些元素之间就会出现空白。而浮动元素会忽略空白节点，互相紧贴。

## IE6和IE7
- Ie67对此属性部分支持。

原文: https://www.w3cplus.com/css/inline-blocks.html © w3cplus.com

