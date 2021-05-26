---
title: go-vscode环境配置
date: 2021-05-26 22:35:07
tags:
  - 环境配置
categories: 
- go
copyright: true
---

### Windows下go+vscode的环境配置

#### 1. 下载安装vscode和 windows下go（https://golang.google.cn/）的默认安装

略

#### 2. 配置Go Env

1. 创建go的工作目录

   > GOPATH（工作目录）
   >
   > -------bin（go的二进制包目录）
   >
   > -------pkg （归档文件目录）
   >
   > -------src （源码文件目录）
   >
   > 在用户环境变量中设置GoPath=工作目录

2. CMD下配置go env

   > 1. go env -w GO111MODULE=on   开启 GO111MODULE   
   > 2. go env -w GOBIN=E:\GOPATH\bin   设置bin目录
   > 3. go env -w GOPROXY=[https://goproxy.cn](https://goproxy.cn/),direct  设置go国内代理(七牛云CDN)

#### 3. vscode配置

1. vscode中go插件安装。
2. 按住Ctrl+Shift+P 输入Go:Install/Update Tools，然后勾选所有，并安装。（前一步如果设置了代理，这里应该会全部成功，否则就会失败）。
3. （可选）安装gocode插件，代码智能提示。go get github.com/stamblerre/gocode

