---
title: STL算法(3)
date: 2019-05-27 17:05:17
tags:
  - STL
categories: 
- STL
copyright: true
---

涉及的算法如下：

mismatch&next_permutation(prev_permutation、is_permutation)&copy&unique&rotate&move

准备工作

```c++
struct Msg {
	int index;
	int msg;
};
//用于打印显式
void PrintVec(vector<Msg>::iterator iter_begin, vector<Msg>::iterator iter_end) {
	for (auto iter = iter_begin; iter != iter_end; iter++) {
		if (iter != iter_begin) {
			cout << "--";
		}
		cout << iter->index << ":" << iter->msg;
	}
	cout << endl << endl;
}
//打印INT数组
void PrintVecInt(vector<int>::iterator iter_begin, vector<int>::iterator iter_end) {
	for (auto iter = iter_begin; iter != iter_end; iter++) {
		if (iter != iter_begin) {
			cout << "--";
		}
		cout << *iter;
	}
	cout << endl << endl;
}
```

<!--more-->
生成数据（所有的排序针对index，msg只是为了标识稳定与否）

```c++
	vector<Msg> v_msg;
	//vector<Msg> v_msg_sec;
	Msg temp_msg;
	int count = 10; 
	srand((int)time(0));
	for (int i = 0; i < count; i++) {
		temp_msg.index = rand() % 100;
		temp_msg.msg = i;
		v_msg.push_back(temp_msg);
		//if (rand() % 2 == 0 && i != 0) {
		//	temp_msg.index = rand() % 100;
		//	temp_msg.msg = i;
		//	v_msg_sec.push_back(temp_msg);
		//}
	}
```

- mismatch

参数：前两个参数表示第一个序列（前后），第三四个参数表示第二个序列，第五个参数可选，谓词（使用==操作符）

作用：两个序列是否匹配，如果不匹配，返回不匹配的位置。

返回值：pair对象，包含两个迭代器，first成员是指向第一个序列中的迭代器，second成员指向第二个。

```C++
	vector<Msg> v_copy(v_msg);
	auto iter_change = v_copy.begin() + 5;
	iter_change->index = 100;
	::stable_sort(v_msg.begin(), v_msg.end(), [](const Msg& m1, const Msg& m2) {return m1.index < m2.index; });
	auto pair_match = ::mismatch(v_msg.begin(), v_msg.end(), v_copy.begin(), v_copy.end(), [](const Msg& m1, const Msg& m2) {return m1.index == m2.index; });
	cout << pair_match.first->index << ":" << pair_match.first->msg << endl;
	cout << pair_match.second->index << ":" << pair_match.second->msg << endl;
```

输出：

> 	4:9
> 	100:9
> 	v_copy:2:7--3:6--4:0--4:4--4:5--100:9--5:2--6:1--6:3--6:8
> 	v_msg :2:7--3:6--4:0--4:4--4:5--4:9--5:2--6:1--6:3--6:8

- next_permutation&prev_permutation

参数：前两个参数表示一个序列，第三个参数：谓词 表示所需的排列方式（更大或更小）。这两个通过谓词的改变可以实现同一个功能（最大或者最小队列）。

作用：求一个序列的最大序列或最小序列。

返回值： `bool` 表示根据谓词是否生成新的序列，如果根据谓词无法生成更大或更小序列时，返回 `false`

```C++
	vector<int> v_int = { 2,3,1,6 };
	//for (auto i = v_int.begin(); i != v_int.end() - 1; i++) {
		//::iter_swap(i, ::min_element(i, v_int.end(), [](const int& m1, const int& m2) {return m1 < m2; }));
	//}
	int i = 1;
	do 
	{
		cout <<"line:"<< i++ << ",v_int:";
		PrintVecInt(v_int.begin(), v_int.end());
	} while (::prev_permutation(v_int.begin(), v_int.end(), [](const int& m1, const int& m2) {return m1 < m2; }));
	//} while (::next_permutation(v_int.begin(), v_int.end(), [](const int& m1, const int& m2) {return m1 < m2; }));
```

输出略（这里结果集较大，序列越长，得到最大或最小序列的次数越多）

- is_permutation

参数：前两个是第一个序列，第三四个表示第二个序列，第五个参数表示谓词即判断条件（操作符==）。

作用：判断第一个序列是否是第二个序列的一个排列，

返回值： `bool` 

```c++
	vector<Msg> v_check(v_msg); 
	::next_permutation(v_msg.begin(), v_msg.end(), [](const Msg& m1, const Msg& m2) {return m1.index < m2.index; });
	temp_msg.index = 100;
	v_msg.push_back(temp_msg);
	bool res = ::is_permutation(v_msg.begin(), v_msg.end(), v_check.begin(), v_check.end(), [](const Msg& m1, const Msg& m2) {return m1.index == m2.index; });
	cout << res << endl;
```

输出：

> 0  
>
> //temp_msg.index = 100;
>
> 1

- copy_n

参数：第一个表示原地址（迭代器），第二个表示个数，第三个表示目的位置

作用：从源容器复制指定个数的元素到目的容器中。

返回值：迭代器，指向目的位置最后一个被复制元素的后一个位置的迭代器。

PS：这里如果目的位置空间不够程序则会崩溃，同时如果个数为0或者负数，那么算法不做任何处理。

```c++
	vector<Msg> v_copy;
	v_copy.resize(v_msg.size());
	::copy_n(v_msg.begin(), v_msg.size(), v_copy.begin());
	cout << "v_copy:";
	PrintVec(v_copy.begin(), v_copy.end());
```

输出：

> 	v_msg:3:0--1:1--7:2--2:3--6:4--6:5--6:6--0:7--2:8--2:9
> 	v_copy:3:0--1:1--7:2--2:3--6:4--6:5--6:6--0:7--2:8--2:9

- unique

参数：前两个参数表示一个序列，第三个是谓词，即去重规则（操作符==）

作用：去除相邻的重复元素（只保留一个），实际上把重复的元素放到最后了，需要自己对那些元素处理。

返回值：迭代器，指向被处理的重复元素起始的位置。

```c++
	auto end_iter = ::unique(v_msg.begin(), v_msg.end(), [](const Msg& m1, const Msg& m2) {return m1.index == m2.index; });
	cout << "v_msg:";
	PrintVec(v_msg.begin(), v_msg.end());
	v_msg.erase(end_iter, v_msg.end());
```

输出：

> 	v_msg:2:0--0:1--2:2--1:3--1:4--0:5--2:6--3:7--1:8--5:9
> 	v_msg:2:0--0:1--2:2--1:3--0:5--2:6--3:7--1:8--5:9--5:9
> 	v_msg:2:0--0:1--2:2--1:3--0:5--2:6--3:7--1:8--5:9

- rotate

参数：第一个参数是序列的开始迭代器，第二个参数是指向一个新的元素的迭代器，必定在序列之内，第三个参数是序列的结束迭代器。

作用：把第二个迭代器直到最后的部分挪到序列的前面，即`旋转`。

返回值：原始的第一个元素所在的新位置迭代器。

```C++
	auto iter = ::rotate(v_msg.begin(), v_msg.begin() + 3, v_msg.end());
	cout << "v_msg_iter:";
	PrintVec(iter, v_msg.end());
```

输出：

> 	v_msg:3:0--0:1--4:2--6:3--4:4--2:5--4:6--1:7--3:8--0:9
> 	v_msg_iter:3:0--0:1--4:2

- move

  参数：前两个参数表示第一个序列，第三个参数表示目的序列的开始位置。

  作用：将一个序列中的一部分移到另一个序列中。原序列无法保证不变。

  返回值：迭代器，指向最后一个被移动到目的序列的元素的下一个位置。

  ```c++
  	vector<Msg> v_move;
  	v_move.resize(v_msg.size());
  	::move(v_msg.begin() + 3, v_msg.end(), v_move.begin() + 1);
  	cout << "v_move:";
  	PrintVec(v_move.begin(), v_move.end());
  ```

  输出：(例子中原序列没变，但是这里不能保证原序列不变)

  > 	v_msg:7:0--6:1--0:2--0:3--0:4--7:5--6:6--7:7--3:8--6:9
  > 	v_move:0:0--0:3--0:4--7:5--6:6--7:7--3:8--6:9--0:0--0:0
  > 	v_msg:7:0--6:1--0:2--0:3--0:4--7:5--6:6--7:7--3:8--6:9