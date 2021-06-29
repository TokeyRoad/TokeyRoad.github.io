---
title: STL算法(1)
date: 2019-04-29 17:15:34
tags:
  - STL
categories: 
- STL
copyright: ture
---
本次所有的算法如下：

##### stable_sort&partial_sort&nth_element&is_sorted&is_sorted_until&merge&inplace_merge&find_if

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
- stable_sort

  <!--more-->

  第三个参数表示 排序的要求（升降序）以及自定义方法.（稳定的排序）
```C++
::stable_sort(v_msg.begin(), v_msg.end(), [](const Msg& m1, const Msg& m2) {return m1.index < m2.index; });//升序(默认升序)
```
排序前：`v_msg:86:0--42:1--74:2--45:3--55:4--42:5--91:6--15:7--59:8--50:9`
排序后：`v_msg:15:7--42:1--42:5--45:3--50:9--55:4--59:8--74:2--86:0--91:6`

- partial_sort
（first,second,last,func） 部分排序 将从first到last中的数据 按照func的规则  排出first到second区间的数量，剩余部分不管（即剩余部分乱序）
```C++
::partial_sort(v_msg.begin(), v_msg.begin() + 5, v_msg.end(), [](const Msg& m1, const Msg& m2) {return m1.index > m2.index; });//降序(默认升序)
```
排序前：`v_msg:48:0--6:1--23:2--68:3--59:4--23:5--78:6--46:7--75:8--60:9`
排序后：`v_msg:78:6--75:8--68:3--60:9--59:4--6:1--23:2--23:5--46:7--48:0`

- nth_element
(first,n,second,func) 执行之后会使数组（执行后）的第n个元素之前的都比他小（大）之后的都比他大（小），对应升序（降序）
但是 这个元素 前后的两个部分 不一定有序(如果数据量小，不易出现，此处数据量调为50个)
```C++
::nth_element(v_msg.begin(), v_msg.begin() + 15, v_msg.end(), [](const Msg& m1, const Msg& m2) {return m1.index > m2.index; });//降序(默认升序)
```
排序前：
		v_msg:3:0--39:1--57:2--73:3--96:4--70:5--76:6--19:7--72:8--32:9--38:10--54:11--84:12--17:13--51:14
		--86:15--11:16--73:17--36:18--47:19--37:20--7:21--8:22--46:23--0:24--90:25--47:26--31:27--80:28
		--7:29--65:30--30:31--57:32--74:33--91:34--30:35--31:36--93:37--92:38--92:39--69:40--55:41--82:42
		--80:43--92:44--50:45--17:46--13:47--70:48--24:49
排序后：
		v_msg:84:12--92:38--93:37--91:34--96:4--86:15--80:43--80:28--92:44--82:42--90:25--92:39--76:6
		--74:33--73:3--73:17--72:8--70:5--70:48--69:40--65:30--57:2--57:32--55:41--54:11--51:14--50:45
		--47:26--47:19--46:23--39:1--38:10--37:20--36:18--32:9--31:36--31:27--30:31--30:35--3:0--19:7
		--0:24--11:16--7:29--17:13--8:22--17:46--13:47--7:21--24:49

- is_sorted
检查指定的区间 是否有序（按规定的排序） 返回true false
```C++
::sort(v_msg.begin(), v_msg.end(), [](const Msg& m1, const Msg& m2) {return m1.index > m2.index; });
bool res = ::is_sorted(v_msg.begin(), v_msg.end(), [](const Msg& m1, const Msg& m2) {return m1.index > m2.index; });
cout << res << endl << endl;
```
第一行注释 即输出0  反之输出1

- is_sorted_until
在指定的区间 按规定的排序 去查找第一个上边界 如果是有序的  则返回end 否则返回指向上边界的迭代器
上边界（降序）： 5 4 3 3/4 则第三位3为降序的第一个上边界
```C++
auto f_iter = ::is_sorted_until(v_msg.begin(), v_msg.end(), [](const Msg& m1, const Msg& m2) {return m1.index < m2.index; });
```

- merge
合并两个有序数组，并存放到第三个数组（理论上第三个数组可以是前两个之一，但是会出现不可预知的错误），可规定排序规则
merge不会新增元素，所以第三个数组需要提前准备好空间
merge返回值是合并之后指向合并最后一个元素的下一个位置
三个排序规则必须相同
```C++
auto r_iter = ::merge(v_msg.begin(), v_msg.end(), v_msg_sec.begin(), v_msg_sec.end(), v_merge.begin(), [](const Msg& m1, const Msg& m2) {return m1.index < m2.index; });
```
- inplace_merge

```C++
		(first,second,last) [first,second),[second,last) 是两个有序数组， 然后在同一数组内合并
		::sort(v_msg.begin(), v_msg.begin() + 5, [](const Msg& m1, const Msg& m2) {return m1.index > m2.index; });
		::sort(v_msg.begin() + 5, v_msg.end(), [](const Msg& m1, const Msg& m2) {return m1.index > m2.index; });
		cout << "v_msg:";
		PrintVec(v_msg.begin(), v_msg.end());
		::inplace_merge(v_msg.begin(), v_msg.begin() + 5, v_msg.end(), [](const Msg& m1, const Msg& m2) {return m1.index > m2.index; });
```
排序前：`v_msg:69:0--56:2--50:3--29:4--7:1--94:9--62:5--61:8--57:6--40:7`
排序后：`v_msg:94:9--69:0--62:5--61:8--57:6--56:2--50:3--40:7--29:4--7:1`

- find_if
根据第三个参数的规则 来查找所需的对象 返回指向对象位置的迭代器 没有则返回end
```C++
auto iter = ::find_if(v_msg.begin(), v_msg.end(), [&temp_msg](Msg m1) {return m1.index < temp_msg.index; });
```