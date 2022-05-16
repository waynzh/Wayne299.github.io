---
title: 'AJAX'
date: 2021-8-12 13:55
categories: FE
tags: [js基础, ajax]
---

# AJAX简介
- 异步的JS和XML
- 通过AJAX可以在浏览器中向服务器发送异步请求：
- 优点：
	- **无刷新的获取数据**
	- 根据**用户事件**更新页面内容
- 缺点：
	- 没有浏览历史，不能回退
	- 存在跨域问题：a.com不能向b.com发送请求 / 同源可以，既协议、主机、端口一致
	- SEO(搜索引擎优化)不友好：源代码里没有响应体
 
## HTTP
<!-- more -->
### 请求

```
行		GET/POST	URL		HTTP/1.1<协议版本>
头		Host：wayneblog.com
		Cookie: name=wayne
		Content-type: application/x-www-form-urlencoded
		User-Agent: chrome 83
空行
体		username=admin&password=admin
```

### 响应

```
行		HTTP/1.1  200  OK
头		Content-Type: text/html;charset=utf-8
		Content-length: 2048;
		Content-encoding: gzip
空行
体		<HTML>
```

## Express基本使用

```js
// 1. 引入express
const express = require("express");

// 2. 创建应用对象
const app = express();

// 3. 创建路由规则
// request: 对请求的分装
// response: 对响应的分装
app.get("/", (request, response)=>{
	// 设置响应头：跨域
	response.setHeader("Access-Control-Allow-Origin", "*");
})；

// 4. 监听端口启动服务
app.listen(8000, () => {
	response.send("server begin!");
})
```

## 基本使用
### POST设置请求体

```js
const xhr = new XHMHttpRequest();
xhr.open("POST", "http://127.0.0.1:8000/server");
// 设置请求头信息：open后 send前
xhr.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
xhr.send();

// readystate 是 xhr 对象中的属性，表示状态
// 0: default 1: open 2: send 3: 服务器端返回部分结果 4: 返回的所有结果
xhr.onreadystatechange = function() {
	if(xhr.readyState === 4) {
		if(xhr.status >= 200  && xhr.status < 300) {
			// 处理结果 
			result.innerHTML = xhr.response;
		}
	}
}
```

### 响应JSON数据

```js
// Express
const data = {...}
let str = JSON.stringify(data);
response.send(str);

// js
// 1. 手动转化
let data = JSON.parse(xhr.response);

// js
// 2. 自动转换：设置响应体数据的类型
const xhr = new XHMHttpRequest();
xhr.responseType = "json";
...
xhr.response.name
```

### AJAX请求超时

```js
const xhr = new XHMHttpRequest();
// 超时设置2s
xhr.timeout = 2000;
// 超时回调
xhr.ontimeout = function() {...};
// 网络异常回调
xhr.onerror = function() {...}
```

### AJAX手动取消请求：解决重复发送问题

```js
let x = null;
x = new XHMHttpRequest();
if(isSending)   xhr.abort();  // 取消正在发送的
x = new XHMHttpRequest(); // 新建一个请求发送
```

## JSONP
- JSONP利用`<script>`标签跨域，返回一个函数调用，其参数就是data
- 只能`get`

## CORS (Cross-Origin Resource Sharing) 跨域资源共享
```js
response.setHeader("Access-Control-Allow-Origin", "*")
response.setHeader("Access-Control-Allow-Headers", "*")
response.setHeader("Access-Control-Allow-Method", "*")
```