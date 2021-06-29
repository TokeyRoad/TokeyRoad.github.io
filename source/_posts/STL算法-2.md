---
title: STL算法(2)
date: 2019-05-10 14:07:42
tags:
  - STL
categories: 
- STL
copyright: true
---
涉及的算法如下：

##### adjacent_find&search_n&partition&binary_search&equal_range

准备工作

```C++
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
```

生成数据（所有的排序针对index，msg只是为了标识稳定与否）
```C++
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
- adjacent_find

  <!--more-->

  用来搜索序列中两个连续相等的元素，用==操作符来判断比较连续的两个元素
  返回值指向第一对相等的元素的第一个，如果没有就指向end
```C++
	//这里先排一下序 组成测试序列
	::stable_sort(v_msg.begin(), v_msg.end(), [](const Msg& m1, const Msg& m2) {return m1.index < m2.index; });//升序(默认升序)
	auto iter = ::adjacent_find(v_msg.begin(), v_msg.end(), [](const Msg& m1, const Msg& m2) {return m1.index == m2.index; });
	if (iter == v_msg.end()) cout << "0" << endl;
	else cout << iter->index << ":" << iter->msg << endl;
```
输出
`3:3`	
`v_msg:0:9--2:1--3:3--3:4--3:5--3:7--6:2--7:0--7:8--8:6`

- search_n
搜索给定元素的匹配项，它在序列中连续出现了给定的次数。
它的前两个参数是定义搜索范围的正向迭代器，第3个参数是想要查找的第4个元素的连续匹配次数。
```C++
	::stable_sort(v_msg.begin(), v_msg.end(), [](const Msg& m1, const Msg& m2) {return m1.index < m2.index; });//升序(默认升序)
	temp_msg.index = 5;
	auto iter = ::search_n(v_msg.begin(), v_msg.end(), 2, temp_msg, [](const Msg& m1, const Msg& m2) {return m1.index == m2.index; });
	if (iter == v_msg.end()) cout << "0" << endl;
	else cout << iter->index << ":" << iter->msg << endl;
```
输出：
`5:0`	
`v_msg:0:2--0:5--0:9--1:6--2:1--2:3--3:8--5:0--5:7--6:4`

- partition
在序列中分区元素会重新对元素进行排列，所有使给定谓词返回 true 的元素会被放在所有使谓词返回 false 的元素的前面。
前两个参数是定义被分区序列范围的正向迭代器，第三个参数是一个谓词。
满足谓词的元素放前面，不满足放后面，即将所有元素分区返回分界点的迭代器
前后两个分区内的元素相对顺序与原始相对顺序不保证一致（不稳定）
stable_partition与partition功能相同，但是能保证相对顺序不变（稳定）
```C++
	temp_msg.index = 5;
	auto iter = ::partition(v_msg.begin(), v_msg.end(), [&temp_msg](const Msg& m) {return m.index < temp_msg.index; });
	if (iter == v_msg.end()) cout << "0" << endl;
	else cout << iter->index << ":" << iter->msg << endl;
```
输出：
`v_msg:1:0--7:1--2:2--1:3--7:4--3:5--0:6--6:7--0:8--5:9`
`7:4`
`v_msg:1:0--0:8--2:2--1:3--0:6--3:5--7:4--6:7--7:1--5:9`

分区验证
is_partitioned + partition_point
前者前两个参数表示一个序列，第三个参数是验证的谓词
当一个序列满足分区之后，才可使用，返回分区的分界点iter，第一分区[begin,iter),[iter,end)
这里不再列出

分区拷贝
partition_copy
原始序列由前两个参数指定。第3个参数用来确定目的序列的开始位置，它会保存那些使谓词返回 true 的元素。
第4个参数用来确定另一个目的序列的开始位置，它会保存那些使谓词返回 false 的元素。第5个参数是用来分区元素的谓词。

- binary_search
二分查找算法
它会在前两个参数指定范围内搜索等同于第三个参数的元素。指定范围的迭代器必须是正向迭代器而且元素必须可以使用<运算符来比较
支持使用谓词（第四个参数）

lower_bound() 算法可以在前两个参数指定的范围内查找不小于第三个参数的第一个元素，也就是说大于等于第三个参数的第一个元素。前两个参数必须是正向迭代器。
upper_bound() 算法会在前两个参数定义的范围内查找大于第三个参数的第一个元素。对于这两个算法，它们所查找的序列都必须是有序的，而且它们被假定是使用 < 运算符来排序的。
注意：lower_bound是大于等于  upper_bound是大于
```C++
	temp_msg.index = 5;
	::stable_sort(v_msg.begin(), v_msg.end(), [](const Msg& m1, const Msg& m2) {return m1.index < m2.index; });//升序(默认升序)
	bool is_find = ::binary_search(v_msg.begin(), v_msg.end(), temp_msg, [](const Msg& m1, const Msg& m2) {return m1.index == m2.index; });
	if (is_find) cout << "1" << endl;
	else cout << "0" << endl;
	//lower_bound + upper_bound
	auto iter = ::lower_bound(v_msg.begin(), v_msg.end(), temp_msg, [](const Msg& m1, const Msg& m2) {return m1.index < m2.index; });
	if (iter == v_msg.end()) cout << "0" << endl;
	else cout << iter->index << ":" << iter->msg << endl;
	iter = ::upper_bound(v_msg.begin(), v_msg.end(), temp_msg, [](const Msg& m1, const Msg& m2) {return m1.index < m2.index; });
	if (iter == v_msg.end()) cout << "0" << endl;
	else cout << iter->index << ":" << iter->msg << endl;
```
输出：
`v_msg:0:7--2:3--2:5--4:2--5:0--6:1--6:6--6:8--6:9--7:4`
`1`
`5:0`
`6:1`

- equal_range
找出有序序列中所有和给定元素相等的元素。
它的前两个参数是指定序列的两个正向迭代器，第三个参数是要查找的元素。
返回一个pair  first指向不小于目标元素的元素，second指向大于目标元素的元素
```C++
	temp_msg.index = 5;
	::stable_sort(v_msg.begin(), v_msg.end(), [](const Msg& m1, const Msg& m2) {return m1.index < m2.index; });//升序(默认升序)
	auto iter_range = ::equal_range(v_msg.begin(), v_msg.end(), temp_msg, [](const Msg& m1, const Msg& m2) {return m1.index < m2.index; });
	cout << "v_res:";
	PrintVec(iter_range.first, iter_range.second);
```
输出：
`v_msg:0:0--0:6--0:7--0:8--1:3--4:1--4:2--5:4--5:5--6:9`
`v_res:5:4--5:5`

equal
如果两个序列的长度相同，并且对应元素都相等，equal() 算法会返回 true  用==操作符
注意：这里必须对应元素相等
```C++
	vector<Msg> v_copy(v_msg);
	::stable_sort(v_copy.begin(), v_copy.end(), [](const Msg& m1, const Msg& m2) {return m1.index < m2.index; });
	bool res = ::equal(v_msg.begin(), v_msg.end(), v_copy.begin(), [](const Msg& m1, const Msg& m2) {return m1.index == m2.index; });
	cout << res << endl;
```