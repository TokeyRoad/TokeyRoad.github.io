---
title: HTTP请求方法
date: 2019-03-30 21:38:16
categories: 
- 网络协议
copyright: ture
---
### HTTP请求方法

​	常用的HTTP请求方式为GET/POST。

​	根据HTTP标准，HTTP有以下多种请求方式：

​	HTTP1.0定义了三种请求方法：GET、POST和HEAD方法。

​	HTTP1.1新增了五种请求方法：OPTIONS、PUT、DELETE、TRACE和CONNECT方法。
	<!--more-->

1. GET：请求指定的页面信息，并返回实体主体。
2. HEAD：类似于GET请求，只不过返回的响应中没有具体的内容，只是用于获取报头。
3. POST：向指定资源提交数据进行处理请求，数据被包含在请求体中。POST请求可能会导致新的资源的生成或者已有资源的修改。（多用于表单提交）
4. PUT：从客户端向服务器传送的数据取代指定的文档的内容。
5. DELETE：请求服务器删除指定的页面。
6. CONNECT：预留给能够将连接改为管道方式的代理服务器。
7. OPTIONS：允许客户端查看服务器的性能。
8. TRACE：回显服务器收到的请求，主要用于测试或诊断。