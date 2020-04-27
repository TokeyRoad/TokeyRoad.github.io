---
title: vector resize与reserve区别
date: 2020-04-27 14:43:13
tags:
  - C++
  - STL
  - vector
categories: 
- C++
copyright: true
---

std::vector的reserve和resize的区别

> 1. reserve：分配空间，更改capacity但不改变size。
> 2. resize：分配空间，更改capacity也改变size。

#### 函数作用

reserve是容器预留空间，不会真正的去创建数组对象，在创建对象之前，不能引用容器内的元素，当加入新元素的时候使用push_bakc()/insert()函数。

resize是改变容器大小，并且创建对象，因此调用这个函数之后，就可以引用容器内的对象，加入新元素的时候可以使用operator[]操作符，或者引用迭代器来引用元素对象。

#### 函数形式

reserve：一个参数，即需要预留的容器的空间；

resize：两个参数，第一个是容器新的大小，第二个是要加入容器中的新元素，如果这个参数被省略，				那么就调用元素对象的默认构造函数。

#### 例子

```C++
std::vector vec;
/*****reserve*****/
vec.reserve(10);//新元素还没有构造
//此时不能用[]访问元素
for(int i = 0; i < 10; i++){
	vec.push_back(i);//新元素这时才构造
}
/*****resize****/
vec.resize(12);
vec[10] = 1;//直接操作新元素
vec[11] = 2;
```

vector在内存中是连续分布的，所以设计的时候会在所有元素外预留一部分空间，否则每次新增元素都需要重新分配，那么效率将会很低。

假如vector中存在1000个元素，两种做法：

```
1. std::vector， 循环调用1000次push_back(将会进行大量次数的内存分配)
2. std::vector(1000), 循环调用1000次push_back(就只会进行几次内存分配)
```

#### 问题

1. 当前容器预留了多大的空间？（不进行内存重分配的情况下，可以容纳多少元素）
2. 怎么重设容器的预留大小？

获取预留空间capacity()

重设预留空间大小reserve()

#### 结论

resize()和reserve()是两回事，前者影响容器中元素个数，后者影响容器预留空间。

> 假设vector vec;  size() = 50 capacity() = 100 那么：
>
> 1. resize(10);//size() == 10  10--49下标的元素被删除,capacity()=100不变，没有内存重分配。
> 2. resize(60);//size()==60    50--59下标用默认构造函数填充,capacity()=100不变，没有内存重分配。
> 3. resize(60,999);//size()==60    50--59下标用999填充,capacity()=100不变，没有内存重分配。
> 4. resize(200);//size()==200    50--199下标用默认构造函数填充,capacity()=200,自动扩容，内存重分配。
> 5. reserve(10);//size()==50  不变，没有元素被删除，capacity()=100，不变，即reserve调用不起作用。
> 6. reserve(60);//size()==50  元素不变，capacity()=100，不变，即reserve调用不起作用。
> 7. reserve(200);//size()==50  元素不变，capacity()=200，扩容，内存重分配。
>
> vector vec(10); //size()==10;capacity()==10;
>
> vec.push_back(999);//size()=11;capacity()=15;//自动扩容，capacity()的结果是不定的，也不一定是15。