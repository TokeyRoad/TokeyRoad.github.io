---
title: 优先队列priority_queue
date: 2020-12-30 09:40:48
tags:
  - priority_queue
categories: 
- C++
copyright: true
---

### priority_queue

priority_queue是个大顶堆容器适配器，提供常数时间级的最大元素查找，对数代价的插入与删除。

成员函数有 判空(empty)、容量(size)、栈顶元素(top)、压栈(push)、出栈(pop)等。

**头文件** 

#include<queue>

**声明方式**

1. 默认

   ```C++
priority_queue<int> q;	//添加元素之后，元素从大到小的顺序出队
   //等价于 priority_queue<int, vector<int>, less<int>> q; 
   priority_queue<int, vector<int>, greater<int>> q;	//元素从小到大的顺序出队
   ```
   
2. 自定义优先级：

   ```C++
   //比较函数return true 表示前者的优先级低于后者
   //第一种写法：
   struct cmp {
   	operator bool ()(int x, int y) {
   		return x > y;//最小值优先
   	}
   }
   priority_queue<int, vector<int>, cmp> q;
   //第二种写法：
   auto cmp = [](int x, int y) { return x > y; };
   priority_queue<int, vector<int>, decltype(cmp)> q(cmp);
   ```

3. 结构体声明方式：

   ```C++
   struct node {
   	int x;
   	string data;
   	bool operator< (const node& a) const {
   		return this.x > a.x;
   	}
   };
   priority_queue<node> q;	//排序规则由 operator< 中的实现决定
   ```

4. 注意：

   结构体比较时通过自定义operator< 操作符来比较元素中的优先级。

   <!--more-->
   
   比较函数return true 表示前者的优先级低于后者

**代码实现**

```C++
template<typename T> void print_queue(T&q) {
	while (!q.empty()) {
		std::cout << q.top() << " ";
		q.pop();
	}
	std::cout << std::endl;
}

struct node {
	node(int sid) {
		id = sid;
	}
	int id;
	std::string data;
	bool operator< (const node& a) const {
		return this->id > a.id;//最小值优先
	}
};

struct cmp_int1 {
	bool operator() (int& a, int& b) {
		return a > b;//最小值优先
	}
};

struct cmp_int2 {
	bool operator() (int& a, int& b) {
		return a < b;//最大值优先
	}
};

int test_Priority_queue() {
	std::priority_queue<int> q;	//从大到小
	for (int n : {1, 8, 5, 6, 3}) {
		q.push(n);
	}
	std::cout << "default: ";
	print_queue(q);

	std::priority_queue<int, std::vector<int>, std::greater<int> > q2;//从小到大
	for (int n : {1, 8, 5, 6, 3}) {
		q2.push(n);
	}
	std::cout << "greater: ";
	print_queue(q2);

	//比较函数return true 表示前者的优先级低于后者
	auto cmp = [](int left, int right) { return left > right; };
	std::priority_queue<int, std::vector<int>, decltype(cmp)> q3(cmp);//从小到大
	for (int n : {1, 8, 5, 6, 3}) {
		q3.push(n);
	}
	std::cout << "lambda: ";
	print_queue(q3);

	//自定义比较函数
	std::priority_queue<int, std::vector<int>, cmp_int1> q4;//从小到大
	for (int n : {1, 8, 5, 6, 3}) {
		q4.push(n);
	}
	std::cout << "cmp_int1: ";
	print_queue(q4);

	std::priority_queue<int, std::vector<int>, cmp_int2> q5;//从小到大
	for (int n : {1, 8, 5, 6, 3}) {
		q5.push(n);
	}
	std::cout << "cmp_int2: ";
	print_queue(q5);

	//自定义结构体
	std::priority_queue<node> q6;
	node n1(4), n2(7), n3(1), n4(9);
	q6.push(n1);
	q6.push(n2);
	q6.push(n3);
	q6.push(n4);
	std::cout << "node operator<: ";
	while (!q6.empty()) {
		std::cout << q6.top().id << " ";
		q6.pop();
	}
	std::cout << std::endl;

	return 0;
}

/*
output
default: 8 6 5 3 1
greater: 1 3 5 6 8
lambda: 1 3 5 6 8
cmp_int1: 1 3 5 6 8
cmp_int2: 8 6 5 3 1
node operator<: 1 4 7 9
*/
```

