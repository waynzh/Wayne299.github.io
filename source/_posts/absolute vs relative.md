---

title: absolute vs relative
date: 2019-11-16
category: 前端
tags: css 
---

# div布局中absolute和relative的区别
## 在正常流中的位置
### relative
 
- 定位为relative的元素脱离正常的文本流中，但其在文本流中的位置依然存在。如图1：

![relative定位](http://images.51cto.com/files/uploadimg/20100909/1528450.jpg)
<!--more-->

- 黄色背景的层定位为relative，红色边框区域为其在正常流中的位置。在通过top、left对其定位后，从灰色背景层(下一个div)的位置可以看出其**正常位置依然存在**。

### absolute

- 定位为absolute的层脱离正常文本流，但与relative的区别是其在正常流中的位置不在存在。如图2：

![absolute定位](http://images.51cto.com/files/uploadimg/20100909/1528451.jpg)

- 可以看到，在将黄色背景层定位为absolute后，灰色背景层(下一个div)自动补上，**脱离正常文本流**。

## 相对于父元素
### relative
	
- relative定位的层总是相对于其**最近的父元素**，无论其父元素是何种定位方式。如图3：
  
![relative父元素](http://images.51cto.com/files/uploadimg/20100909/1528452.jpg)
  
- 红色背景层为relative定位，其直接父元素绿色背景层为默认的static定位。红色背景层的位置为相对绿色背景层top、left个20元素。
  
### absolute
		
- absolute定位的层总是相对于其**最近的定义为absolute或relative的父层**，而这个父层并**不一定是其直接父层**，如图4:

![absolute父元素](http://images.51cto.com/files/uploadimg/20100909/1528453.jpg)
	
- 红色背景层相对的元素变为定位方式为absolute或relative的黄色背景层。如果其父层中都未定义absolute或relative，则其将相对body进行定位。