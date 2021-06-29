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

必须拷贝文件：
 ├──_config.yml
 ├── theme
 ├── scaffolds #文章模板
 ├── package.json #说明使用哪些包
 ├── .gitignore #限定在提交的时候哪些文件可以忽略
 └── source

<!--more-->

（1）讨论下哪些文件是必须拷贝的：首先是之前自己修改的文件，像站点配置_config.yml，theme文件夹里面的主题，以及source里面自己写的博客文件，这些肯定要拷贝的。除此之外，还有三个文件需要有，就是scaffolds文件夹（文章的模板）、package.json（说明使用哪些包）和.gitignore（限定在提交的时候哪些文件可以忽略）。其实，这三个文件不是我们修改的，所以即使丢失了，也没有关系，我们可以建立一个新的文件夹，然后在里面执行hexo init，就会生成这三个文件，我们只需要将它们拷贝过来使用即可。总结：_config.yml，theme/，source/，scaffolds/，package.json，.gitignore，是需要拷贝的。

（2）再讨论下哪些文件是不必拷贝的，或者说可以删除的：首先是.git文件，无论是在站点根目录下，还是主题目录下的.git文件，都可以删掉。然后是文件夹node_modules（在用npm install会重新生成），public（这个在用hexo g时会重新生成），.deploy_git文件夹（在使用hexo d时也会重新生成），db.json文件。其实上面这些文件也就是是.gitignore文件里面记载的可以忽略的内容。总结：.git/，node_modules/，public/，.deploy_git/，db.json文件需要删除。

### 6.git bash

在本文件夹下git bash,运行npm install

### 7.安装部署插件(可选)

npm install hexo-deployer-git --save

### 8.测试

此时就可以hexo g && hexo d 测试是否成功了。

 

 

 

 

 