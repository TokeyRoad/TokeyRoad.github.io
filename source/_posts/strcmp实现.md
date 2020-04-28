---
title: strcmp实现
date: 2020-04-28 09:54:39
tags:
  - C++
categories: 
- C++
copyright: true
---

#### strcmp函数实现以及分析

##### strcmp函数原型

> int strcmp(const char* str1, const char* str2);
>
> str1 < str2 返回负值或-1
>
> str1 == str2 返回0
>
> str1 > str2 返回正值或1

##### 实现原理

该函数实际上是对字符的ASCII码进行比较，实现方案：从前往后依次比较两个字符串的字符，如果不相等，就停止比较并返回结果，如果相等就继续，直到其中一个字符串遇到结束符'\0'为止。

##### Linux源码

```C++
int strcmp(const char* str1, const char* str2){
	unsigned char c1, c2;
	while(1){
		c1 = *str1++;
		c2 = *str2++;
		if(c1 != c2)	return (c1 < c2) ? -1 : 1;
        if(!c1)	break;
	}
    return 0;
}
```

##### 注意

这个函数体内没有判断参数为NULL时的情况，所以当传入NULL时程序会崩溃，<string>中的strcmp也会崩溃