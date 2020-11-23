---
title: 重装系统之后hexo博客恢复
date: 2020-11-22 21:57:11
tags:
  - hexo
  - git
categories: 
- 环境配置
copyright: true
---

### 1.安装node.js和git

这个不用多说，直接下载安装就行了。

### 2.配置 git 个人信息，生成新的 ssh 密钥：

git config --global user.name "xxxxxx"
 git config --global user.email "xxxxxx"
 ssh-keygen -t rsa -C "xxxxxxxx(邮箱)"

### 3.添加公钥

在用户文件夹.ssh 文件里面把公钥复制出来粘贴到github个人设置的ssh位置。

### 4.安装hexo

 npm install hexo-cli -g

### 5.删除博客文件夹文件，保留部分

打开原来的博客文件夹,只需保留_config.yml，theme/，source/，scaffolds/，package.json，.gitignore 这些项目，删除其他的文件。

### 6.git bush

在本文件夹下git bush,运行npm install

### 7.安装部署插件

npm install hexo-deployer-git --save

### 8.测试

此时就可以hexo g && hexo d 测试是否成功了。

 

 

 

 

 