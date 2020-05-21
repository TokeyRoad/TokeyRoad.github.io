---
title: 线程资源同步对象-condition_variable
date: 2020-05-20 14:41:58
tags:
  - condition_variable
categories: 
- C++
copyright: true
---

C++ 11提供了std::condition_variable这个类 代表条件变量，与Linux系统原生的条件变量一样，同时还提供了等待条件变量的一系列wait方法(wait/wait_for/wait_until)，发送信号使用notify方法(notify_one/notify_all)，使用std::condition_variable时需要绑定到std::unique_lock或std::lock_guard对象。

> C++ 11中 std::condition_variable 不再需要显示调用方法初始化和销毁

```C++
#include <iostream>
#include <mutex>
#include <shared_mutex>
#include <thread>
#include <stdio.h>
#include <list>


class Task {
public:
	Task(int id) {
		m_taskID = id;
	}
	~Task() = default;

	void DoTask() {
		std::cout << "do task,taskID:" << m_taskID << ", threadID:" << std::this_thread::get_id() << std::endl;
	}
private:
	int m_taskID;
};

std::mutex mutex;
std::condition_variable cv;
std::list<Task*> tasks;

void produce_thread() {
	Task* pTask = NULL;
	int taskID = 0;
	while (1) {
		pTask = new Task(taskID);
		{
			std::lock_guard<std::mutex> lg(mutex);
			tasks.push_back(pTask);
			std::cout << "produce a task,taskID:" << taskID << ",threadID:" << std::this_thread::get_id() << std::endl;
		}
		cv.notify_all();
		taskID++;
		std::this_thread::sleep_for(std::chrono::seconds(1));
	}
	return;
}

void consume_thread() {
	Task* pTask = NULL;
	int taskID = 0;
	while (1) {
		std::unique_lock<std::mutex> guard(mutex);
		while (tasks.empty()) {
			cv.wait(guard);
		}
		pTask = tasks.front();
		tasks.pop_front();
		if (pTask == NULL) {
			continue;
		}
		pTask->DoTask();
		delete pTask;
		pTask = NULL;
	}
	return;
}


int main() {
	std::thread consume1 = std::thread(consume_thread);
	std::thread consume2 = std::thread(consume_thread);
	std::thread consume3 = std::thread(consume_thread);
	std::thread consume4 = std::thread(consume_thread);
	std::thread consume5 = std::thread(consume_thread);

	std::thread produce1 = std::thread(produce_thread);
	//std::thread produce2 = std::thread(produce_thread);
	//std::thread produce3 = std::thread(produce_thread);

	consume1.join();
	consume2.join();
	consume3.join();
	consume4.join();
	consume5.join();
	produce1.join();
	//produce2.join();
	//produce3.join();

	system("pause");
	return 0;
}
//output:
produce a task,taskID:0,threadID:19444
do task,taskID:0, threadID:6344
produce a task,taskID:1,threadID:19444
do task,taskID:1, threadID:6352
produce a task,taskID:2,threadID:19444
do task,taskID:2, threadID:4836
produce a task,taskID:3,threadID:19444
do task,taskID:3, threadID:6352
produce a task,taskID:4,threadID:19444
do task,taskID:4, threadID:6344
produce a task,taskID:5,threadID:19444
do task,taskID:5, threadID:4836
produce a task,taskID:6,threadID:19444
do task,taskID:6, threadID:13436
produce a task,taskID:7,threadID:19444
do task,taskID:7, threadID:4836
    .........
```

