---
title: effective_C++
date: 2017-06-03 14:47:42
tags: [c_plus]
---

# 绪论

0.除非有需要使用隐式转换的构造函数，否则构造函数声明为`explicit`

```
class Person
{
public:
	explicit Person(std::string name = "None", unsigned int age = 0);
	...
};
```

# 让自己习惯`C++`

1.把C++视为语言联邦：

+ C
+ 面相对象
+ 泛型编程
+ STL

2.尽量以`const`，`enum`，`inline`替换`#define`

```
class Person
{
private:
	static const int number = 1; //有初值的声明
	int array[number];
};

const int Person::number; //定义
```

使用`enum hack`

```
class Person
{
private:
	enum { number = 1 }; //number成为1的记号
	int array[number];
};
```

3.尽量使用`const`

当const和non-const成员函数有等价的实现时，可以用non-const版本调用const版本，避免代码重复，
但是不能反过来调用，因为可能带来风险。

4.确定对象使用前已被初始化

类构造函数进入函数体之前用成员初始化列表初始化

为避免初始化顺序问题，使用local static代替non-local static

# 构造、析构、赋值运算

5.了解`C++`默默编写并调用哪些函数

如果含有引用成员，或者const成员，必须自己定义拷贝赋值操作符

如果基类将拷贝赋值运算符声明为private，编译器将拒绝为其派生类生成拷贝赋值运算符

6.若不想使用编译器自动生成的函数，就应该明确拒绝

将其声明为`private`

7.为多态基类声明`virtual`析构函数

给基类添加virtual析构函数，当销毁基类指针指向的派生类对象时，正确销毁整个对象

如果不希望基类被实例化，可以把析构函数声明为纯虚函数

析构函数运作方式是先最深的开始，然后往基类调用，所以需要先实现基类析构函数的定义

```
class A
{
public:
	virtual ~A() = 0;
};

A::A() {} //定义纯虚函数的定义
```

如果设计目的不是具有多态性质，就不该声明virtual函数

8.别让异常逃离析构函数

析构函数绝对不要抛出异常，如果析构函数调用的函数可能抛出异常，析构函数应该捕捉任何异常，吞下或结束程序

9.绝不在构造和析构过程中调用`virtual`函数

在析构函数和构造函数不要调用virtual函数，因为这类调用不会下降到派生类

10.令`operate=`返回一个`*this`的引用

```
Widget& operator=(const Widget& rhs)
{
	return *this;
}
```

11.在`operator=`处理自我赋值

通过调整语句顺序实现自我赋值和异常安全

```
Widget& operator=(const Widtget& rhs)
{
	int* pOrig = pb;  //先备份
	pb = new int(*rhs.pb);  //创建副本
	delete pOrig; //删除原先的
	return *this;
}
```

12.复制对象时勿忘其每一个成分

拷贝函数应该确保复制对象内的所有成员变量以及所有基类成分

不要尝试以一个拷贝函数实现另外一个拷贝函数，应该将共同机能放在第三个函数中，由另外两个拷贝函数调用

# 资源管理

13.以对象管理资源

RAII对象：构造函数中获取资源，析构函数中释放资源

常被使用的是`auto_ptr`和`shared_ptr`

14.在资源管理类中小心拷贝行为

对资源进行引用计数

```
class Lock
{
public:
	explicit Lock(Mutex *pm)   //以某个mutex初始化shared_ptr
		: mutexPtr(pm, unlock)  //以unlock作为删除器
	{
		lock(mutexPtr.get()); //用以返回智能指针内部的原始指针
	}
private:
	std::shared_ptr<Mutex> mutexPtr;
}
```

复制RAII对象必须一并复制它所管理的资源

创建的RAII复制行为是：抑制复制行为、使用引用计数法

15.在资源管理中提供对原始资源的访问

对原始资源的访问可能经由显式转换或隐式转换，显示比较安全，隐式比较方便

16.成对使用 `new`和`delete`时要采取相同形式

如果在调用`new`时使用`[]`，必须字对应调用`delete`时也使用`[]`

17.以独立语句将newed对象置入智能指针

以独立语句将newed对象置入智能指针，如果不这样做，一旦异常被抛出，可能发生资源泄漏

```
processWidget(std::shared_ptr<Widget>(new Widget), priority())
```

应该使用独立的语句

```
std::shared_ptr<Widget> pw(new Widget);
processWidget(pw, priority());
```

# 设计与声明

18.让接口容易被正确使用，不易被误用

19.设计`class`犹如设计`type`


20.宁以pass-by-reference-to-const替换pass-by-value

可以避免copy构造函数被调用

可以避免出现对象切割问题，例如

```
void printName(window w);
```

如果传入的参数是w的派生类，对象将被切割成window类。

如果使用reference-to-const

```
void printName(const window& w);
```

传入的参数可以正确的表示该类型

21.返回对象时，不能返回reference

22.将成员变量声明为private

23.已non-member、 non-friend替换member函数

将多个头文件放在多个文件，但是隶属于同一个命名空间，
有利于便利函数的扩充。

while(1); //待续
