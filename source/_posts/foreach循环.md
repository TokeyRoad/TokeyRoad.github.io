---
title: foreach循环
date: 2020-05-18 14:36:54
tags:
  - foreach
categories: 
- C++
copyright: true
---

#### for-each

C++ 11之后才开始支持for-each语法。

示例：

```C++
	std::vector<std::string> v_str;
	v_str.push_back("aaa");
	v_str.push_back("bbb");
	v_str.push_back("ccc");
	v_str.push_back("ddd");
	for (auto iter : v_str) {
		std::cout << iter.c_str() << std::endl;
	}
```

##### 注意

> 1. for-each中的迭代器类型与数组或集合中的元素类型完全一致，而stl容器中的迭代器是类型的取地址类型即指针，因此对于上面的例子中，iter是string类型；如果用stl迭代器，那就是指向string的指针。
> 2. for-each对于复杂数据类型，迭代器是原始数据的拷贝，而不是引用，在这个过程中会额外调用构造函数的开销，必要的时候可以使用`auto& iter` 而不是 `auto iter` 这样就是原始数据的引用了。

示例

```C++
	std::vector<std::string> v_str;
	v_str.push_back("aaa");
	v_str.push_back("bbb");
	v_str.push_back("ccc");
	v_str.push_back("ddd");
	for (auto iter : v_str) {
		iter = "hello";
	}
	for (auto iter : v_str) {
		std::cout << iter.c_str() << std::endl;
	}

//output
aaa
bbb
ccc
ddd
```

修改后

```C++
	std::vector<std::string> v_str;
	v_str.push_back("aaa");
	v_str.push_back("bbb");
	v_str.push_back("ccc");
	v_str.push_back("ddd");
	for (auto& iter : v_str) {
		iter = "hello";
	}
	for (auto iter : v_str) {
		std::cout << iter.c_str() << std::endl;
	}

//output
hello
hello
hello
hello
```

#### 自定义对象使用for-each（Range-based）

for-each的lterator类型必须支持如下三种操作:

> operator++操作，即自增，可以自增返回下一个迭代子的位置
>
> operator!=操作，即判不等
>
> operator* 操作，即解引用

示例：

```C++
#include <iostream>
#include <vector>

template<typename T, size_t N>
class A {
public:
	A() {
		for (size_t i = 0; i < N; ++i) {
			m_elements[i] = i;
		}
	}
	~A() {}
	T* begin() {
		return m_elements + 0;
	}
	T* end() {
		return m_elements + N;
	}
private:
	T m_elements[N];
};

//这里迭代子的类型是T* 本身就支持operator++ 和 operator!=操作，这里就不再实现

int main() {
	A<int, 10> arr;
	for (auto iter : arr) {
		std::cout << iter << std::endl;
	}
	system("pause");
	return 0;
}


```

