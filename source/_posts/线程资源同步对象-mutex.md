---
title: 线程资源同步对象-mutex
date: 2020-05-20 09:34:48
tags:
  - mutex
categories: 
- C++
copyright: true
---

在C++ 11新标准中新增了线程资源同步对象：std::mutex 和 std::condition_variable 。

##### std::mutex 系列

C++ 11/14/17中提供了如下mutex系列类型：

|        互斥量         |  版本  |                     作用                     |
| :-------------------: | :----: | :------------------------------------------: |
|         mutex         | C++ 11 |                最基本的互斥量                |
|      timed_mutex      | C++ 11 |              有超时机制的互斥量              |
|    recursive_mutex    | C++ 11 |                可重入的互斥量                |
| recursive_timed_mutex | C++ 11 | 结合timed_mutex和recursive_mutex特点的互斥量 |
|  shared_timed_mutex   | C++ 14 |          具有超时机制的可共享互斥量          |
|     shared_mutex      | C++ 17 |                 共享的互斥量                 |

这个系列的对象均提供了加锁(lock)、尝试加锁(trylock) 和 解锁(unlock) 的方法。

为了避免死锁， std::mutex.lock() 和 std::mutex.unlock() 需要成对使用，同时C++11提供了一些互斥量管理的封装方法，避免忘记unlock而造成死锁。

| 互斥量管理  |  版本  |          作用          |
| :---------: | :----: | :--------------------: |
| lock_guard  | C++ 11 | 基于作用域的互斥量管理 |
| unique_lock | C++ 11 |  更加灵活的互斥量管理  |
| shared_lock | C++ 14 |    共享互斥量的管理    |
| scope_lock  | C++ 17 | 多互斥量避免死锁的管理 |

示例：

```C++
void func(){
	std::lock_guard<std::mutex> lg(mymutex);
	//保护的资源操作
}
```

注意：mymutex的声明周期必须比函数func的作用域长。

错误示例：

```C++
//错误写法
void func(){
	std::mutex m;
	std::lock_guard<std::mutex> lg(mymutex);
	//保护的资源操作
}
```

这样写是错误的，当跳出func作用域时，m已经失效，这时lg析构解锁的时候，会出现未定义的行为。

此外，如果一个std::mutex对象已经调用lock()方法，再次调用时,会出现未定义的行为(Windows)。

```C++
#include <iostream>
#include <mutex>

int main() {
	std::mutex m;
	m.lock();
	m.lock();	//程序在这里崩溃
	//bool re = m.try_lock();
	//std::cout << re << std::endl;	//output 0
	m.unlock();
	system("pause");
	return 0;
}
```

因此，对于一个已经调用lock()方法再次调用lock()方法的做法是错误的。

##### std::shared_mutex

std::shared_mutex底层实现主要原理是操作系统提供的读写锁，在存在多个线程对公共资源读，少部分线程对公共资源写的情况下，std::shared_mutex 比 std::mutex效率更高。

std::shared_mutex提供了lock_shared()和unlock_shared()方法 获取读锁和写锁，写锁通常被称为 排他锁(Exclusive Locking)，读锁常被称为 共享锁(Shared Locking)。

同时C++ 新标准中引入了 std::unique_lock 和 std::shared_lock 用于进入作用域自动加锁，离开作用域自动解锁，前者用于写锁，后者用于读锁。

示例：

```C++
#include <iostream>
#include <mutex>
#include <shared_mutex>
#include <thread>
#include <stdio.h>

class shared_mutex_counter {
public:
	shared_mutex_counter() = default;
	~shared_mutex_counter() = default;

	int get() {
		std::shared_lock<std::shared_mutex> sl(_shared_mutex);
		return _value;
	}

	void increse() {
		std::unique_lock<std::shared_mutex> sl(_shared_mutex);
		++_value;
	}

	void reset() {
		std::unique_lock<std::shared_mutex> sl(_shared_mutex);
		_value = 0;
	}

private:
	std::shared_mutex _shared_mutex;
	//共享数据
	int _value = 0;
};

#define LOOP_COUNT 5000000
#define THREAD_COUNT 8

void test_shared_mutex() {
	shared_mutex_counter shared_counter;
	int result = 0;

	auto readr = [&shared_counter, &result]() {
		for (int i = 0; i < LOOP_COUNT; ++i) {
			result = shared_counter.get();
		}
	};

	clock_t start = clock();
	auto writer = [&shared_counter]() {
		for (int i = 0; i < LOOP_COUNT; ++i) {
			shared_counter.increse();
		}
	};

	std::thread** tarray = new std::thread*[THREAD_COUNT];
	for (int i = 0; i < THREAD_COUNT; ++i) {
		tarray[i] = new std::thread(readr);
	}
	std::thread* tw = new std::thread(writer);

	for (int i = 0; i < THREAD_COUNT; ++i) {
		tarray[i]->join();
	}
	tw->join();
	clock_t end = clock();
	printf("[test_shared_mutex] thread_count:%d\n", THREAD_COUNT);
	printf("count_value:%d, cost:%dms, result:%d", shared_counter.get(), end - start, result);
}


int main() {
	test_shared_mutex();
	system("pause");
	return 0;
}

//output
[test_shared_mutex] thread_count:8
count_value:5000000, cost:5030ms, result:2522449
```

另外一个mutex的测试案例不再列出，跟shared_mutex类似。但是两者效率差距较大。

如果条件允许，可以根据使用场景，用shared_mutex替换mutex，可以提高很大的效率。

