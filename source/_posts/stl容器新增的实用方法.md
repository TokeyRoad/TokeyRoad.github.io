---
title: stl容器新增的实用方法
date: 2020-05-19 11:44:34
tags:
  - stl
categories: 
- C++
copyright: true
---

#### emplace系列函数

先上一段常用的代码：

```C++
#include <iostream>
#include <list>
#include <vector>

class Test {
public:
	Test(int a, int b) {
		m_a = a;
		m_b = b;
		std::cout << "Test constructed" << std::endl;
	}
	~Test() {
		std::cout << m_a << " Test destructed" << std::endl;
	}

	Test(const Test& t) {
		if (this == &t) {
			return;
		}
		this->m_a = t.m_a;
		this->m_b = t.m_b;
		std::cout << m_a << " Test copy-constructed" << std::endl;
	}
private:
	int m_a;
	int m_b;
};

int main() {
	std::vector<Test> vec;
	vec.reserve(20);//，提前分好内存 避免vector的内存重分配
	for (int i = 0; i < 3; ++i) {
		Test t(i, i + 1);
		vec.push_back(t);
	}
	system("pause");
	return 0;
}
//output
Test constructed
0 Test copy-constructed
0 Test destructed
Test constructed
1 Test copy-constructed
1 Test destructed
Test constructed
2 Test copy-constructed
2 Test destructed
```

在这个过程中，我们的目的是为了产生一个对象数组，为了产生3个对象,调用了3*3次，这个过程执行过程：创建对象t调用构造函数--->调用对象t的拷贝构造函数放入集合--->调用析构函数。

<!--more-->

C++11 提供了一个新的方法代替emplace_back

```C++
	std::vector<Test> vec;
	vec.reserve(20);//，提前分好内存 避免vector的内存重分配
	for (int i = 0; i < 3; ++i) {
		vec.emplace_back(i, i+1);
	}
//output
Test constructed
Test constructed
Test constructed
```

emplace操作被称为 “原位构造元素”

同理的方法 如下:

|   原方法    | C++11 改进方法 |         含义         |
| :---------: | :------------: | :------------------: |
| push/insert |    emplace     | 指定位置原位构造元素 |
| push_front  | emplace_front  |   首部原位构造元素   |
|  push_back  |  emplace_back  |   尾部原位构造元素   |

#### std::map的try_emplace 与 insert_or_assign方法

由于map中的key是唯一的，因此在开发中经常遇到往map中插入数据之前需要先验证key是否存在，C++17中map提供了一个try_emplace方法，该方法会检测指定的key是否存在，函数签名如下：

```C++
template <class... Args>
pair<iterator, bool> try_emplace(const key_type& k, Args&&... args);

template <class... Args>
pair<iterator, bool> try_emplace(key_type&& k, Args&&... args);

template <class... Args>
iterator try_emplace(const_iterator hint, const key_type& k, Args&&... args);

template <class... Args>
iterator try_emplace(const_iterator hint, key_type&& k, Args&&... args);
```

其中k表示需要插入的key，args不定参数表示构造value对象所需的参数列表，hint表示插入位置。

前两种形式中，返回值是std::pair<T1,T2>，其中T2是个bool表示，当前插入操作成功与否，T1是一个迭代器，如果插入成功，则指向插入位置的元素迭代器，如果失败，则指向相同key元素的迭代器。 

后两种不常用，在map中插入时很少关心插入位置。

示例：

```C++
#include <iostream>
#include <string>
#include <map>

int main() {
	std::map<std::string, std::string> m;
	m.emplace("aaa", "ONE");
	m.emplace("bbb", "TWO");
	m.emplace("ccc", "THREE");
	m.emplace("ddd", "FOUR");

	auto res = m.try_emplace("eee", "FIVE");
	std::cout << res.first->second.c_str() << " " << res.second << std::endl;
	std::cout << "map size:" << m.size() << std::endl;

	for (auto iter : m) {
		std::cout << iter.first << iter.second << " ";
	}
	std::cout << std::endl;

	auto resf = m.try_emplace("aaa", "SIX");	//auto -> std::pair<std::map<std::string, std::string>::iterator, bool>
	std::cout << resf.first->second.c_str() << " " << resf.second << std::endl;

	for (auto iter : m) {
		std::cout << iter.first << iter.second << " ";
	}
	std::cout << std::endl;

	auto it = m.try_emplace(m.begin(), "fff", "SEVEN");	////auto -> std::map<std::string, std::string>::iterator
	std::cout << it->first << it->second << std::endl;

	for (auto iter : m) {
		std::cout << iter.first << iter.second << " ";
	}
	std::cout << std::endl;

	return 0;
}
//output
FIVE 1
map size:5
aaaONE bbbTWO cccTHREE dddFOUR eeeFIVE
ONE 0
aaaONE bbbTWO cccTHREE dddFOUR eeeFIVE
fffSEVEN
aaaONE bbbTWO cccTHREE dddFOUR eeeFIVE fffSEVEN
```

try_emplace是map中指定的key存在就失败，不存在就插入，还有另一个方法insert_or_assign 当key存在时就更新value，不存在时就插入。

insert_or_assign函数签名：

```C++
template <class M>
pair<iterator, bool> insert_or_assign(const key_type& k, M&& obj);

template <class M>
pair<iterator, bool> insert_or_assign(key_type&& k, M&& obj);

template <class M>
iterator insert_or_assign(const_iterator hint, const key_type& k, M&& obj);

template <class M>
iterator insert_or_assign(const_iterator hint, key_type&& k, M&& obj);
```

使用示例

```C++
	std::map<std::string, std::string> m;
	m.emplace("aaa", "ONE");
	m.emplace("bbb", "TWO");
	m.emplace("ccc", "THREE");
	m.emplace("ddd", "FOUR");
	for (auto iter : m) {
		std::cout << iter.first << iter.second << " ";
	}
	std::cout << std::endl;
	
	m.insert_or_assign("aaa", "EIGHT");
	for (auto iter : m) {
		std::cout << iter.first << iter.second << " ";
	}
	std::cout << std::endl;
//output
aaaONE bbbTWO cccTHREE dddFOUR
aaaEIGHT bbbTWO cccTHREE dddFOUR
```

