---
title: HTTP协议(1)
date: 2019-03-28 21:56:48
categories: 
- 网络协议
copyright: ture
---
### **HTTP协议**（1）

#### 简介

​	HTTP协议是Hyper Text Transfer Protocol（超文本传输协议）的缩写，从万维网（World Wide Web）传输超文本到本地浏览器的协议。

#### 工作原理

​	HTTP协议是工作在C/S架构上的，浏览器作为HTTP客户端通过URL向HTTP服务端（WEB服务器）发送数据请求。

​	常用的WEB服务器：Apache，Nginx，IIS服务器等。

​	服务器接收到请求之后，向客户端响应信息即回复。

​	HTTP默认端口号80，可自己在WEB服务器上修改。

##### 		HTTP注意事项

1. HTTP是无连接的：即每次只能处理一个请求，每次服务端收到请求之后就立马处理并回复，收到客户端的应答之后就断开连接，这样极大的减少了传输时间。

2. HTTP是媒体独立的：即只有客户端和服务端知道怎么处理传输的内容、以及内容的格式，任何格式的数据都可以通过HTTP协议传输，客户端和服务端用指定的type去接收和处理。

3. HTTP是无状态的：无状态是指协议对于事务处理没有记忆能力，意味着如果后续处理需要前面连接的数据，那么这些数据需要重新传输，这样就导致了连接时的数据量增大，另一方面，在服务器不需要先前信息时传输较快。

#### HTTP消息结构

​	HTTP使用统一资源标识符（Uniform Resource Identifiers,URI）来传输数据和建立连接。

##### 	客户端请求消息

​	客户端发送一个HTTP请求到服务端的请求消息格式：请求行（request line）、请求头部（header） 、空行和请求数据四部分组成。下图为请求报文的一般格式。

![clientRequest](HTTP协议-1/clientRequest.png)

##### 服务器响应消息

​	HTTP响应也是由四个部分组成，分别是：状态行、消息报头、空行和响应正文。

![serverResponse](HTTP协议-1/serverResponse.jpg)

#### 实例

以下是使用GET来传递数据的实例：

客户端请求：

```
GET /hello.txt HTTP/1.1
User-Agent: curl/7.16.3 libcurl/7.16.3 OpenSSL/0.9.7l zlib/1.2.3
Host: www.example.com
Accept-Language: en, mi
```

服务端响应：

```
HTTP/1.1 200 OK
Date: Mon, 27 Jul 2009 12:28:53 GMT
Server: Apache
Last-Modified: Wed, 22 Jul 2009 19:15:56 GMT
ETag: "34aa387-d-1568eb00"
Accept-Ranges: bytes
Content-Length: 51
Vary: Accept-Encoding
Content-Type: text/plain
```

输出结果：

```
Hello World! My payload includes a trailing CRLF.
```