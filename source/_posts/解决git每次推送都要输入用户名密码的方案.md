---
title: 解决git每次推送都要输入用户名密码的方案
date: 2020-11-22 23:08:41
tags:
  - git
categories: 
- 环境配置
copyright: true
---

打开命令行，输入以下命令:

```git
git config --global credential.helper store
```

执行完毕会在在当前系统用户文件夹下生成一个名为.git-credentials的文件，如：C:\Users\Administrator\ .git-credentials，再次提交代码时，输入密码后会将用户名密码以明文的方式保存在其中。 

