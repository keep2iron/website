---
title: 大话Http
date: 2019-03-29 10:17:14
tags:
---

> HyperText Transfer Protocol 超文本传输协议

* 超文本：在电脑中显示的，可以指向其他链接的文本


## 报文

HTTP报文是在HTTP应用程序之间发送的数据块。这些数据块以一些文本形式的*元信息(meta-infomation)*开头，这些信息描述了报文的内容及含义，后面跟着可选的数据部分。

#### 报文的语法（Message Syntax）
HTTP报文是简单的格式化数据块。每条报文都包含一条来自客户端的请求，或者一条来自服务器的响应。它们由三个部分组成：对报文进行描述的*起始行（start line）*、包含属性的*首部（Header）块，以及可选的包含数据的主题（Body）部分。*
````
HTTP-message = start-line
			   *( header-field CRLF)
				CRLF
				[ message-body]
````

#### 报文分类与格式
#### 请求示例
![](https://i.imgur.com/eP4NtPS.png)