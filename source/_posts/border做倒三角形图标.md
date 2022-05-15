---
title: 'border的部分应用'
date: 2019-11-30 19:20:43
category: FE
tags: css
---
# border的部分应用
## border做倒三角形图标
```css
a:after {
	display: block;
	content: "";
	/*美人尖*/
	border: 5px solid;
	border-color: #b00000 transparent transparent; /*上 左右 下*/
	width: 0; 
}
```
- `:after` 选择器是在a后插入内容
- 因为**边框是楔形**，所以当内容为空白,**只显示边框**时，形成**美人尖**。
