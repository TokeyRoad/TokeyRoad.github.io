---
title: 原子操作类-atomic
date: 2020-05-22 11:29:15
tags:
  - atomic
categories: 
- C++
copyright: true
---

C++ 11 新标准中提供了对整形变量原子操作的相关库，即std::atomic,这是个模板类型：（同时支持跨平台）

```C++
template<class T>
struct atomic;
```

注意：

```C++
#include <iostream>
#include <atomic>
#include <stdio.h>
int main() {
	std::atomic<int> ato_int;
	ato_int = 1;
    //std::atomic<int> ato_int = 1;
	printf("%d \n", (int)ato_int);

	++ato_int;
	printf("%d \n", (int)ato_int);
	system("pause");
	return 0;
}
```

以上代码在win和linux中都可以运行，但是如果这样：

```C++
	//std::atomic<int> ato_int;
	//ato_int = 1;
    std::atomic<int> ato_int = 1;
```

就只能在win下运行，linux编译无法通过。

错误原因是std::atomic的拷贝构造函数是默认=delete 禁止编译器生成的 g++遵循了，但是win上的vc++没有遵循。

```C++
atomic& operator=(const atomic&) = delete;
```

|                         方法名                          |                           方法说明                           |
| :-----------------------------------------------------: | :----------------------------------------------------------: |
|                        operator=                        |                         存储值到对象                         |
|                          store                          |              原子的以非原子对象替换原子对象的值              |
|                          load                           |                    原子的获取原子对象的值                    |
|                        exchange                         |          原子的替换原子对象的值并获得它先前替换的值          |
|   compare_exchange_weak<br />compare_exchange_strong    | 原子的比较原子对象非原子参数的值。若相等则交换，不相等则加载 |
|                        fetch_add                        |         原子的将参数加到原子对象的值，并返回先前的值         |
|                        fetch_sub                        |         原子的将原子对象的值减去参数，并返回先前的值         |
|                        fetch_and                        |      原子的进行参数和原子对象逐位与，并获得先前保存的值      |
|                        fetch_or                         |      原子的进行参数和原子对象逐位或，并获得先前保存的值      |
|                        fetch_xor                        |     原子的进行参数和原子对象逐位异或，并获得先前保存的值     |
|   operator++ operator++(int) operator- operator-(int    |                      原子值增加或者减一                      |
| operator+= operator-= operator&= operator\|= operator^= |               =加减，=与原子值进行逐位与或异或               |

