---
title: char*赋值给std::string的陷阱
date: 2020-04-29 10:47:31
tags:
  - C++
categories: 
- C++
copyright: true
---

##### char*赋值给std::string的一些陷阱

1. 将char * 赋值给std::string如果不指定长度，则会默认以\0截断（ASCII码值为0）；如果指定长度超过char* 的长度，用std::cout系列的函数输出时，会把不属于char*之后的内存值打印出来，而对于printf系列的函数打印时，遇到\0会被截断，因而不能完全显示。这点在打印日志时，这类字符串需要注意。

2. 如果是单个字符和一个字符串赋值给std::string 写法是有区别的：对于char，数目是第一个参数，对于char*，数目是第二个参数。
   > string(const char* s, size_t n);
   >
   > string(size_t n, char c);
   
   假定pstr是一个字符串，那么要写成string(pstr,n)；如果pstr是一个字符，那么要写成string(n,pstr)，而此时string(pstr, n)是一个错误的写法，可能会导致你的程序产生莫名其妙的问题，因为如果pstr是一个负值，负值转换成无符号整数size_t类型，n将非常大，会导致构造字符串时length非常大导致std::string 构造时抛出异常。

