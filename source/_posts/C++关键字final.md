---
title: C++关键字final
date: 2019-04-29 11:38:01
categories: 
- C++
copyright: ture
---
## C++ 11的新特性之final##

final是说明符而不是关键字，这意味着可以将它用作标识符(override同样)

说明符final，用于阻止派生类覆盖特定的虚方法。或者是阻止一个类成为基类。
	<!--more-->
```C++
class Base {
	virtual void f();
};
		
class A :Base {
	void f() final;		//override  也是最后一次override
	void g() final;		//Error 不能使用final修饰符声明非虚函数
};
		
class B final :A {
	void f() override;	//父类已经声明为final 不能被override
};
		
class C :B {			//class B已声明为final 不能被继承
	//
};```