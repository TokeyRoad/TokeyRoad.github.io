---
title: const用法总结
date: 2019-07-01 10:32:27
tags:
  - C++
  - Const
categories: 
- C++
copyright: true
---

简单总结一下const的用法。

#### 一、const修饰普通类型的变量。

```c++
const int a = 7;
int b = a;	//正确
a = 8;		//错误，const修饰的参数为常量，不可修改
```

a被 const修饰之后，被编译器识别为一个常量，常量不能被赋值。

<!--more-->

简单实例：

```c++
#include<iostream>

int main(void){
    const int a = 7;
    int *p = (int*)&a;
    *p = 8;
    std::cout << a;		//下断点，调试
    //这里会输出几呢?
    //输出：7
    system("pause");	
    return 0;
}
```

这个例子中，const 修饰 a，无法直接对a的值进行修改，但是通过*p直接对a地址内的值进行了赋值。

下断点调试时，会发现在 `*p = 8` 之后，a的值就变为了8。

但是程序最后输出的还是7。

从上述实例中发现，编译器对const修饰的a进行了优化，一直认为a的值为定义的7，当对它的地址的值修改之后，编译器并没有去修改它，认为它还是常量7，如果后续有用到a的操作可能会产生意想不到的结果。

如果不想让编译器对它的优化，在const前面添加 volatile关键字。

volatile关键字和const是相反的，不会被编译器优化，a的值也就会被改变了。

```c++
const int a = 7;
改为：
volatile const int a = 7;
```

这时，结果就会输出期望值 8。

#### 二、const修饰指针变量。

const修饰指针有三种情况：

- const修饰指针指向的内容，其内容不可变。
- const修饰指针，指针不可变。
- const修饰指针和指针指向的内容，两者都不可变。

第一种：

```c++
const int *p = 8;
//const 位于 * 之前，则表示指针指向的内容不可变（左定值），即 8不可修改。
```

第二种：

```c++
int a = 8;
int * const p = &a;
*p = 9;	//正确，修改了a地址的值 8->9
int b = 7;
p = &b;	//错误
//const在 * 之后，表示指针不可变（右定向），即p只能指向a地址，不能修改。
```

第三种：

```c++
int a = 8;
const int * const p = &a;
//第一种和第二种的合并
```

"左定值，右定向，const修饰不变量"

#### 三、const参数传递和参数返回值。

##### const修饰函数参数可分为三种情况：

第一种：值传递的const修饰，一般这种情况不需要修饰，函数会自动产生临时变量复制实参值。

```c++
void Test(const int a){
    ++a;//错误
}
```

第二种：当const修饰指针时，可以防止指针被意外篡改。

```c++
void Test(int * const a){
    *a = 9;
    //int b = 5;
    //a = &b;	//错误
}

int main(void){
    int a = 8;
    Test(&a);
    cout << a; // a = 9
    return 0;
}
```

第三种：自定义类型的参数传递（常用类型用的相对较少），需要临时构造对象复制参数，比较浪费时间，因此采用const加引用传递的方式。

```c++
class Test{
public:
    Test(int para){
        m_Para = para;
    }
    int getPara() const {	//表示返回值不可变  下文有提到
        return m_para;
    }
private:
    int m_Para;
};

void TestFun(const Test& t){
    std::cout << t.getPara();
}

int main(void){
    Test t(8);
    TestFun(t);
    system("pause");
    return 0;
}
```

##### const修饰函数的返回值也有三种情况：

第一种：const修饰内置类型的返回值，修饰与不修饰返回值作用一样。

```c++
const int CTest(){
    return 1;
}

int Test(){
    return 0;
}

int main(void){
    int a = CTest();
    int b = Test();
    cout << a << " " << b;
    return 0;
}
```

第二种：const修饰自定义类型的作为返回值，此时返回的值不能作为左值使用，不能被修改。

第三种：const修饰返回的指针或者引用，是否返回一个指向const的指针，取决于函数的作用。

#### 四、const修饰类成员函数

const修饰类成员函数，其目的是防止成员函数修改被调用对象的值，如果不想修改一个调用对象的值，所有的成员函数都应该声明为const成员函数。

**重点：** const关键字不能与static关键字同时使用，因为static关键字修饰静态成员函数，静态成员函数不含有this指针，即不能实例化，const成员函数必须具体到某一实例。

如果有个成员函数想修改对象中的某一个成员，其他的不允许修改，这时就可以使用mutable关键字修饰这个成员，被mutable修饰的成员变量可以在const成员函数中被修改。

```c++
class Test{
public:
    void t()const{
        ++a;	//错误 
        ++b;	//正确
    }
private:
    int a;
    mutable int b;
};
```
