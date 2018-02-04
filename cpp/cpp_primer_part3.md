---
title: C++ Primer
date: 2016-11-26 18:49:04
tags: [c_plus]
---


# 拷贝控制

## 拷贝赋值和销毁

### 拷贝构造函数

13.5

```
//注意，这里省略了 std::
class HasPtr
{
public:
    HasPtr(const string &s = string()) : 
        ps(new string(s)), i(0) {}
    HasPtr(const HasPtr &hp);
    void print() {cout << *ps << endl;}
    void change(const string s) {*ps = s;}
private:
    string *ps;
    int i;
};

HasPtr::HasPtr(const HasPtr &hp) :
    ps(new string(*hp.ps)),
    i(hp.i) {}

int main () 
{
    HasPtr a("hello");
    HasPtr b = a;
    a.print(); //hello
    b.print(); // hello
    a.change("world");
    a.print(); // world
    b.print(); // hello
    b.change("asd");
    a.print(); // world
    b.print(); //asd
} 
```

### 拷贝赋值运算符

13.8

重载赋值运算符，返回类型是当前对象的引用

```
class HasPtr
{
public:
    HasPtr(const string &s = string()) : 
        ps(new string(s)), i(0) {}
    HasPtr(const HasPtr &hp);
    HasPtr &operator=(const HasPtr &hp);
    void print() {cout << *ps << endl;}
    void change(const string s) {*ps = s;}
private:
    string *ps;
    int i;
};

HasPtr::HasPtr(const HasPtr &hp) :
    ps(new string(*hp.ps)),
    i(hp.i) {}

//重载赋值运算符，返回类型是当前对象的引用
HasPtr &HasPtr::operator=(const HasPtr &hp)
{
    //防止hp和ps指向同一对象
    auto newp = new string(*hp.ps); //存到临时变量
    delete ps; //释放旧内存
    ps = newp; //拷贝到本地
    i = hp.i;
    return *this;
}

int main () 
{
    HasPtr a("hello");
    HasPtr b;
    b = a;
    a.print(); //hello
    b.print(); // hello
    a.change("world");
    a.print(); // world
    b.print(); // hello
    b.change("asd");
    a.print(); // world
    b.print(); //asd
} 
```

### 析构函数

初始化对象非static成员，释放对象使用的资源，销毁非static数据成员

没有返回值，不接收参数

```
class HasPtr
{
public:
    HasPtr(const string &s = string()) : 
        ps(new string(s)), i(0) {}
    ~HasPtr() {delete ps;} //析构函数
    HasPtr(const HasPtr &hp);
    HasPtr &operator=(const HasPtr &hp);
    string *val(){return ps;}
    void print() {cout << *ps << endl;}
    void change(const string s) {*ps = s;}
private:
    string *ps;
    int i;
};

HasPtr::HasPtr(const HasPtr &hp) :
    ps(new string(*hp.ps)),
    i(hp.i) {}

HasPtr &HasPtr::operator=(const HasPtr &hp)
{
    auto newp = new string(*hp.ps);
    delete ps;
    ps = newp;
    i = hp.i;
    return *this;
}

//返回一个指针，指向string
string *fun()
{
    HasPtr hp("hello");
    return hp.val();
}

int main () 
{
    string *p = fun(); //p指向一个已经被释放的string
    cout << *p << endl; //报错，因为指针已经悬空
} 
```

### 阻止拷贝

13.18

利用`static`数据成员给每个对象生成唯一的递增的数据成员

```
class Employee
{
public:
    Employee(const string s) : name(s),num(count)
        {count = count + 1;}
    void print() {cout << num << " : " << name << endl;}
private:
    string name;
    int num;
    static int count; //内部声明
};

//static外部定义，注意没有static关键字
int Employee::count = 0;

int main () 
{
    Employee e1("a");
    Employee e2("b");
    Employee e3("c");
    e1.print(); // 0 : a
    e2.print(); // 1 : b
    e3.print(); // 2 : c
} 
```

## 拷贝控制和资源管理ka

### 引用计数

13.27

计数器保存在动态内存中

```
class HasPtr
{
public:
    HasPtr(const string &s = string()) : 
        ps(new string(s)), i(0), use(new size_t(0)) {++ *use;}
    ~HasPtr(); 
    HasPtr(const HasPtr &hp);
    HasPtr &operator=(const HasPtr &hp);
    string *val(){return ps;}
    void print() {cout << *use << " " << *ps << endl;}
    void change(const string s) {*ps = s;}
private:
    string *ps;
    int i;
    size_t *use;
};
//拷贝初始化，自增计数器
HasPtr::HasPtr(const HasPtr &hp) :
    ps(new string(*hp.ps)),
    i(hp.i), use(hp.use) {++*use;}
//析构函数，自减计数器，计数器为0释放内存
HasPtr::~HasPtr()
{
    if (-- *use == 0)
    {
        delete ps;
        delete use;
    }
}
//拷贝赋值运算符，右值自增，左值自减计数器
HasPtr &HasPtr::operator=(const HasPtr &hp)
{
    ++ *hp.use; //先自增，防止指向的是同一对象
    if(-- *use == 0)
    {
        delete ps;
        delete use;
    }
    ps = hp.ps;
    i = hp.i;
    use = hp.use;
    return *this;
}
//查看引用计数
int main () 
{
    HasPtr a1("hello");
    a1.print(); // 1
    HasPtr a2("world");
    a1.print(); // 1
    a2.print(); // 1
    HasPtr a3 = a1;
    a1.print(); // 2
    a2.print(); // 1
    a3.print(); // 2
    a3 = a2;
    a1.print(); // 1
    a2.print(); // 2
    a3.print(); // 2
} 
```

## 交换操作

13.31

在赋值运算中调用swap

```
class HasPtr
{
friend void swap(HasPtr &hp1, HasPtr &hp2);
public:
    HasPtr(const string &s = string()) : 
        ps(new string(s)), i(0), use(new size_t(0)) {++ *use;}
    ~HasPtr(); 
    HasPtr(const HasPtr &hp);
    HasPtr &operator=(HasPtr hp);
    bool operator<(const HasPtr &hp) const;
    string *val(){return ps;}
    void print() {cout << *ps << endl;}
    void change(const string s) {*ps = s;}
private:
    string *ps;
    int i;
    size_t *use;
};
//拷贝初始化，同上
HasPtr::HasPtr(const HasPtr &hp) :
    ps(new string(*hp.ps)),
    i(hp.i), use(hp.use) {++*use;}
//析构函数，同上
HasPtr::~HasPtr()
{
    if (-- *use == 0)
    {
        delete ps;
        delete use;
    }
}
//使用swap重写拷贝赋值操作
HasPtr &HasPtr::operator=(HasPtr hp)
{  
     //副本在脱离代码块时，自动调用析构函数释放内存
    swap(*this, hp); //当前对象于一个副本交换
    return *this;
}
//< 运算符，在调用sort时需要
bool HasPtr::operator<(const HasPtr &hp) const
{
    cout << "operator < ..." << endl;
    return *this -> ps < *hp.ps;
}
//swap操作
inline
void swap(HasPtr &hp1, HasPtr &hp2)
{
    //如果类没有自己的swap调用标准库的swap
    //如果类有自己的swap，不会隐藏自己的swap
    using std::swap; 
    swap(hp1.ps, hp2.ps);
    swap(hp1.i, hp2.i);
    cout << "swaped..." << endl;
}

int main () 
{
    HasPtr a1("hello");
    HasPtr a2("world");
    HasPtr a3("byebye");
    vector<HasPtr> vhp{a1, a2, a3};
    sort(vhp.begin(), vhp.end()); //排序
    /*
        输出：这里调用了三次比较，四次swap拷贝赋值
        operator < ...
        operator < ...
        swaped...
        operator < ...
        swaped...
        swaped...
        swaped...
    */
} 
```

# 重载运算和类型转换

## 重载运算符

14.2

重载Sales_data的输入、输出、加法和复合赋值运算符
重载相等运算符

```
class Sales_data
{
friend std::istream &operator>>(std::istream &is, Sales_data &sd);
friend std::ostream &operator<<(std::ostream &os, const Sales_data &sd);
friend Sales_data operator+(const Sales_data &sd1, const Sales_data &sd2);
friend bool operator==(const Sales_data &sd1, const Sales_data &sd2);
friend bool operator!=(const Sales_data &sd1, const Sales_data &sd2);
//...省略...
public:
    Sales_data &operator+=(const Sales_data&);
//...省略...
};

//成员函数
inline
Sales_data& Sales_data::operator+=(const Sales_data &sd)
{
    unit += sd.unit;
    revenue += sd.revenue;
    return *this;
}

//非成员函数
istream &operator>>(istream &is, Sales_data &sd)
{
    double price = 0;
    is >> sd.name >> sd.unit >> price;
    if(is) //判断流是否发生错误
        sd.revenue = price * sd.unit;
    else //如果错误，赋值为默认状态
        sd = Sales_data();
    return is;
}
ostream &operator<<(ostream &os, const Sales_data &sd)
{
    os << sd.getname() << " " << sd.unit << " "
        << sd.revenue << " " << sd.avg_price() << " num: " << sd.num;
    return os;
}
Sales_data operator+(const Sales_data &sd1, const Sales_data &sd2)
{
    Sales_data sum = sd1;
    sum += sd2;
    return sum;
}
bool operator==(const Sales_data &sd1, const Sales_data &sd2)
{
    return sd1.name == sd2.name &&
        sd1.unit == sd2.unit &&
        sd1.revenue == sd2.revenue;
}
bool operator!=(const Sales_data &sd1, const Sales_data &sd2)
{
    return !(sd1 == sd2);
}

int main () 
{
    Sales_data sd1, sd2;
    cin >> sd1; //输入
    cin >> sd2;
    Sales_data sd3 = sd1 + sd2; //加
    sd3 += sd1; //复合赋值
    cout << sd3 << endl; //输出
} 
```

14.18

比较运算符根据实际情况选择是否添加
给StrBlob添加比较运算符

```
class StrBlob
{
friend bool operator<(const StrBlob &sb1, const StrBlob &sb2);
//...
};
//非成员函数
bool operator<(const StrBlob &sb1, const StrBlob &sb2)
{
    auto b1 = sb1.data -> begin(), e1 = sb1.data -> end();
    auto b2 = sb2.data -> begin(), e2 = sb2.data -> end();
    while (b1 != e1 && b2 != e2)
    {
        if (*b1 != *b2)
            return *b1 < *b2;
        ++b1;
        ++b2;
    }
    return sb1.data -> size() < sb2.data -> size();
}

int main () 
{
    StrBlob b1{"abc","afd"};
    StrBlob b2{"abc","afd","asd"};
    cout << (b1 < b2) << endl; // 1
} 
```

14.26

下标运算符通常定义两个版本，一个返回普通的引用，另一个是常量成员并且返回常量引用

```
class StrBlob
{
//...
    string &operator[](size_t n);
    const string &operator[](size_t n) const;
//...
};
//成员函数
//普通版本
string &StrBlob::operator[](size_t n)
{
    return (*data)[n];
}
//常量版本
const string &StrBlob::operator[](size_t n) const
{
    return (*data)[n];
}

int main () 
{
    StrBlob b{"abc","afd"};
    const StrBlob a(b);
    a[1] = "world"; //错误，常量引用只读
    b[1] = "hello";
    cout << b[1] << endl; // hello
} 
```

14.30

给StrBlobPtr添加自增、自减和解引用运算符

```
//先申明
class StrBlobPtr;
class StrBlob
{
friend class StrBlobPtr; //申明为友元
public:
    typedef vector<string>::size_type size_type;
    //构造函数
    StrBlob():data(make_shared<vector<string>>()){}
    StrBlob(initializer_list<string> ils)://多个字符串参数
        data(make_shared<vector<string>>(ils)) {}
    //成员函数
    StrBlobPtr begin();
    StrBlobPtr end();
    size_type size() const {return data -> size();}
    bool empty() const {return data -> empty();}
    void push_back(const string &t) {data -> push_back(t);}
    void pop_back() {check(0, "empty"); data -> front();}
    string& front() {check(0, "empty"); return data -> front();}
    string& back() {check(0, "empty"); return data -> back();}

private:
    shared_ptr<vector<string>> data;//vec的指针
    //检测异常
    void check(size_type i, const string &msg) const 
        {if (i >= data -> size()) throw out_of_range(msg);}
};

class StrBlobPtr
{
public:
    //构造函数
    StrBlobPtr() : curr(0) {}
    StrBlobPtr(StrBlob &a, size_t sz=0):
        wptr(a.data), curr(sz) {} //用shared_ptr初始化弱指针
    //成员函数
    string &operator*() const; //解引用
    string *operator->() const;
    StrBlobPtr &operator++(); //递增
    StrBlobPtr &operator--(); //递增
    StrBlobPtr operator++(int i); //递增
    StrBlobPtr operator--(int i); //递增
private:
    //检查vector是否被释放，索引是否合法
    shared_ptr<vector<string>> check(size_t sz, const string &msg) const;
    weak_ptr<vector<string>> wptr; //一个弱指针
    size_t curr; //位置
};

//返回一个指向第一个元素的StrBlobPtr
StrBlobPtr StrBlob::begin()
{
    return StrBlobPtr(*this);
}
//最后一个元素后一个位置
StrBlobPtr StrBlob::end()
{
    auto ret = StrBlobPtr(*this, data -> size());
    return ret;
}
shared_ptr<vector<string>> 
StrBlobPtr::check(size_t sz, const string &msg) const
{
    auto ret = wptr.lock(); //如果不存在返回空
    if (!ret)
        throw runtime_error("unbond StrBlob");
    if (sz >= ret -> size())
        throw out_of_range(msg);
    return ret;
}

//解引用
string &StrBlobPtr::operator*() const
{
    auto p = check(curr, "dereference past end"); //检查索引
    return (*p) [curr]; //取vector元素
}
string *StrBlobPtr::operator->() const
{
    return & this -> operator*(); //返回解引用结果的地址
}

//前置版本
StrBlobPtr &StrBlobPtr::operator++()
{
    check(curr, "increment past end of StrBlobPtr"); //检查索引
    ++curr; //自增移动
    return *this;
}
StrBlobPtr &StrBlobPtr::operator--()
{
    --curr;//自减移动，如果为0，产生无效下标
    check(curr, "increment past end of StrBlobPtr"); //检查索引
    return *this;
}

//后置版本用一个不被使用的 int 形参与前置版本区分
//后置版本返回的值应该与之前的值保持一致
//返回的是值，而不是引用
StrBlobPtr StrBlobPtr::operator++(int i)
{
    StrBlobPtr ret = *this;
    ++*this; //自增移动
    return ret; //返回原值
}
StrBlobPtr StrBlobPtr::operator--(int i)
{
    StrBlobPtr ret = *this;
    --*this;
    return ret;
}

int main () 
{
    StrBlob b{"a","aa","aaa"};
    StrBlobPtr bp = b.begin(), be = b.end();
    cout << *bp++ << endl; //a
    cout << *++bp << endl;//aaa
} 
```

# OOP

## 定义基类和派生类

15.3

定义一个基类

```
class myquote
{
public:
    myquote() = default;
    myquote(const std::string &bookname, double source_price) :
        name(bookname), price(source_price) {}
    std::string getname() const {return name;}
    //需要派生类自己定义的函数声明为虚函数
    virtual double getprice() const {return price;}
    virtual ~myquote() = default; //基类需要定义一个虚解析函数
private: //私有成员，用户和派生类都不能访问
    std::string name;
protected: // 受保护的成员，派生类可以访问，用户不能访问
    double price;
};
```

定义一个打印函数

```
//这里的mq可以是基类或者派生类
void print_total(ostream &os, MyQuote &mq, size_t num)
{
    double total_price = mq.getprice() * num;
    os << mq.getname() << " : " << total_price << endl;
}
```

15.5

定义一个派生类

```
class CutQuote : public MyQuote
{
public:
    CutQuote() = default;
    CutQuote(const std::string &bookname, double source_price, int cut) : 
        MyQuote(bookname, source_price), discount(cut) {}
    double getprice() const override {return price/discount;} //重写虚函数
private:
    int discount; //新增数据成员表示折扣
};
```

15.11

定义一个输出成员值的debug()函数

```
class MyQuote
{
public:
    //....
    virtual void debug()
    { //输出自己的元素
        std::cout << "name : " << name << std::endl 
            << "price : " << price << std::endl;
    }
//...
};

class CutQuote : public MyQuote
{
public:
    //....
    void debug()
    { //显式调用基类的debug函数，否则会陷入无限递归
        MyQuote::debug(); 
        std::cout << "discount : " << discount << std::endl;
    }
//...
};
```

15.15

抽象基类

```
class MyQuote
{
public:
    MyQuote() = default;
    MyQuote(const std::string &bookname, double source_price) :
        name(bookname), price(source_price) {}
    std::string getname() const {return name;}
    virtual double getprice() const {return price;}
    virtual ~MyQuote() = default;
private: //私有成员，用户和派生类都不能访问
    std::string name;
protected: // 受保护的成员，派生类可以访问，用户不能访问
    double price;
};

//继承基类的抽象基类
class DiscQuote : public MyQuote
{
public:
    DiscQuote() = default;
    DiscQuote(const std::string &bookname, double source_price, int cut) : 
        MyQuote(bookname, source_price), discount(cut) {}
    double getprice() const = 0; //声明为纯虚函数
protected:
    int discount; //新增数据成员表示折扣
};

//从抽象基类继承
class CutQuote : public DiscQuote
{
public:
    CutQuote() = default;
    CutQuote(const std::string &bookname, double source_price, int cut) :
        DiscQuote(bookname, source_price, cut) {}
    //覆盖抽象基类的getprice()函数
    double getprice() const override {return price/discount;}
};

void print_total(std::ostream &os, MyQuote &mq, std::size_t num);
```

## 构造函数和派生类

15.26

定义拷贝构造函数

```
class MyQuote
{
public:
    MyQuote() = default;
    MyQuote(const std::string &bookname, double source_price) :
        name(bookname), price(source_price) {}
    std::string getname() const {return name;}
    MyQuote(const MyQuote &mq) : name(mq.name), price(mq.price)
    { //输出表示调用这个拷贝构造函数
        std::cout << "MyQuote's copy.." << std::endl;
    }
    virtual double getprice() const {return price;}
    virtual ~MyQuote() = default;
private: //私有成员，用户和派生类都不能访问
    std::string name;
protected: // 受保护的成员，派生类可以访问，用户不能访问
    double price;
};

class DiscQuote : public MyQuote
{
public:
    DiscQuote() = default;
    DiscQuote(const std::string &bookname, double source_price, int cut) : 
        MyQuote(bookname, source_price), discount(cut) {}
    //先调用基类拷贝构造，然后构造自己的成员
    DiscQuote(const DiscQuote &dq) : MyQuote(dq), discount(dq.discount)
    {
        std::cout << "DiscQuote's copy.." << std::endl;
    }
    double getprice() const = 0; //声明为纯虚函数
protected:
    int discount; //新增数据成员表示折扣
};

class CutQuote : public DiscQuote
{
public:
    CutQuote() = default;
    CutQuote(const std::string &bookname, double source_price, int cut) :
        DiscQuote(bookname, source_price, cut) {}
    // 不用构造自己的成员，向上调用
    CutQuote(const CutQuote &cq) : DiscQuote(cq)
    {
        std::cout << "CutQuote's copy" << std::endl;
    }
    double getprice() const override {return price/discount;}
};

void print_total(std::ostream &os, MyQuote &mq, std::size_t num);

//在主文件调用
int main () 
{
    CutQuote q1("hello world", 10, 3);
    CutQuote  q2(q1);
    print_total(cout, q2, 1);
    /*
        输出说明是自顶而下的构造
        MyQuote's copy..
        DiscQuote's copy..
        CutQuote's copy
        hello world : 3.33333
    */
} 
```

## 继承构造函数

15.27

```
class CutQuote : public DiscQuote
{
public:
    using DiscQuote::DiscQuote; //继承DiscQuote的构造函数
    double getprice() const override {return price/discount;}
};
```

## 容器和继承

15.28

如果在基类类型容器放置派生类对象，派生类那部分会被切掉
因此，在容器中放置（智能）指针而不是对象

```
int main () 
{
    CutQuote q1("hello world", 10, 3);
    MyQuote q2("hello world", 10);
    vector<shared_ptr<MyQuote>> basket;
    //派生类的智能指针隐式转化为基类智能指针
    basket.push_back(make_shared<CutQuote>(q1));
    basket.push_back(make_shared<MyQuote>(q2)); 
    for(auto i : basket)
    {
        print_total(cout, *i, 1);
    }
    /*
        输出：
        hello world : 3.33333
        hello world : 10
    */
} 
```

15.30

编写一个Basket类
如果要改变add_item的接口，直接传入对象，让类自己判断调用什么拷贝方式
需要创建虚拟拷贝

```
class MyQuote
{
//...
    //虚拟拷贝，在程序运行的时候觉得调用哪个类的拷贝函数
    //左值，返回基类指针
    virtual MyQuote *clone() const & {return new MyQuote(*this);} 
    //右值，基类指针
    virtual MyQuote *clone() && {return new MyQuote(std::move(*this));} 
//...
};

class CutQuote : public DiscQuote
{
public:
    using DiscQuote::DiscQuote;
    //左值，派生类指针
    CutQuote *clone() const & {return new CutQuote(*this);}
    //右值，派生类指针
    CutQuote *clone() && {return new CutQuote(std::move(*this));}
    double getprice() const override {return price/discount;}
};

class Basket
{
public:
    void add_item(const MyQuote &mq)
    { //添加到容器内
        items.push_back(std::shared_ptr<MyQuote>(mq.clone()));
    }
    void add_item(MyQuote &&mq)
    { //尽管参数是右值，mq本身却是个左值，所以需要再次调用move
        items.push_back(std::shared_ptr<MyQuote>(std::move(mq).clone()));
    }
    double total_price()
    { //遍历求和
        double sum(0.0);
        for(auto i : items)
        {
            sum += i -> getprice();
        }
        return sum;
    }
private:
    //存放指针的容器
    std::vector<std::shared_ptr<MyQuote>> items;
};

int main () 
{
    CutQuote q1("hello world", 10, 3);
    MyQuote q2("hello world", 10);
    Basket bk;
    bk.add_item(q1);
    bk.add_item(q2);
    cout << bk.total_price() << endl; //13.3333
} 
```

15.39

文件查询程序

```
#ifndef MYSEARCH_H
#define MYSEARCH_H

#include <memory>
#include <string>
#include <map>
#include <vector>
#include <set>
#include <fstream>
#include <iostream>
#include <sstream>

#include "mysearch.h"

class QueryResult;
class TextQuery
{
friend class QueryResult;
public:
    TextQuery(std::ifstream &infile);
    QueryResult query(std::string s) const;
private:
   std::shared_ptr<std::vector<std::string>> ifile; //指向存放每行文本的vector
    std::map<std::string, std::shared_ptr<std::set<int>>> wm; //单词到存放行数的set的映射
};

class QueryResult
{
friend std::ostream &operator<<(std::ostream &os, const QueryResult &qr);
public:
    QueryResult(std::string s, std::shared_ptr<std::set<int>> lp, std::shared_ptr<std::vector<std::string>> fp) : 
        world(s), linep(lp), file(fp){}
    std::set<int>::iterator begin()
    {
        return linep -> begin();
    }
    std::set<int>::iterator end()
    {
        return linep -> end();
    }
    std::shared_ptr<std::vector<std::string>> &get_file()
    {
        return file;
    }
private:
    std::string world;
    std::shared_ptr<std::set<int>> linep;
    std::shared_ptr<std::vector<std::string>> file;
};

//构造函数，在构造函数里储存将单词到行号的隐射
TextQuery::TextQuery(std::ifstream &infile) : ifile(new std::vector<std::string>) //初始化infile
{
    std::string s,world;
    int num(0);
    while(getline(infile, s))
    {
        ++num;
        ifile -> push_back(s);
        std::istringstream line(s);
        while(line >> world)
        {
            if(!wm[world]) //如果set还没有创建
                wm[world].reset(new std::set<int>); //创建一个set
            wm[world] -> insert(num); //行号插入到set
        }
    }
}
//查找
//返回一个QueryResult类
QueryResult TextQuery::query(std::string s) const
{
    std::shared_ptr<std::set<int>> nodata(new std::set<int>); //一个空set
    auto res = wm.find(s); //查找函数的本体，查找map对应字符串的行号set
    if (res == wm.end()) //无内容返回空set
        return QueryResult(s, nodata, ifile);
    else //有内容返回set，访问map 的 second成员
        return QueryResult(s, res -> second, ifile);
}

//用<<操作符打印QueryResult的内容
std::ostream &operator<<(std::ostream &os, const QueryResult &qr)
{
    os << qr.world << ": \n";
    for (auto num:*qr.linep) //对每个指针解引用然后遍历
    {
        os << num << " "; //行号
        os << *(qr.file -> begin() + num - 1) << "\n"; //用迭代器输出vector的行文本
    }
    return os;
}

//一个抽象基类，继承查询类型
class Query_base
{
friend class Query;
protected:
    virtual ~Query_base() = default;
private:
    virtual QueryResult eval(const TextQuery &t) const = 0;
    virtual std::string rep() const = 0;
};

class Query
{
friend Query operator~(const Query &q);
friend Query operator&(const Query &q1, const Query &q2);
friend Query operator|(const Query &q1, const Query &q2);
public:
    //构造一个WordQuery
    Query(const std::string &s);
    QueryResult eval(const TextQuery &t) const
    {
        return q -> eval(t); //虚调用
    }
    std::string rep() const
    {
        return q -> rep(); //虚调用
    }
private:
    Query(std::shared_ptr<Query_base> sq) : q(sq) {}
    std::shared_ptr<Query_base> q;
};

 //继承查询基类
class WordQuery : public Query_base
{
    //默认都是私有成员，由友元Query（用户借口类）访问
    friend class Query;
    //保存要查询的词
    WordQuery(const std::string &s) : query_world(s) {}
    QueryResult eval(const TextQuery &t) const override
    {
        return t.query(query_world); 
    }
    std::string rep() const override {return query_world;}
    std::string query_world; //查询的词
};

inline
Query::Query(const std::string &s) : q(new WordQuery(s)) {}

class NotQuery : public Query_base
{
    friend Query operator~(const Query  &q);
    NotQuery(const Query &q) : query(q) {};
    std::string rep() const override
    { //输出要查询的词，附带~符号和括号
        return "~(" + query.rep() + ")"; 
    }
    QueryResult eval(const TextQuery &t) const override;
    Query query; //保存传递的query参数
};

inline
Query operator~(const Query &q)
{//返回一个shared_ptr
    return std::shared_ptr<Query_base> (new NotQuery(q));
}

class BinaryQuery : public Query_base
{
protected:
    BinaryQuery(const Query &l, const Query &r, std::string s) : 
        lq(l), rq(r), ops(s) {}
    std::string rep() const override
    { //输出查找的词和符号
        return "(" + lq.rep() + " " 
            + ops + " " 
            + rq.rep() + ")";
    }
    //没有覆盖eval()，仍是抽象基类
    Query lq, rq; //左右操作数
    std::string ops; //运算符
};

//&运算和|运算相似
class AndQuery : public BinaryQuery
{
    friend Query operator&(const Query &q1, const Query &q2);
    AndQuery(const Query &q1, const Query &a2) : 
        BinaryQuery(q1, a2, "&") {}
    QueryResult eval(const TextQuery &t) const;
};

inline
Query operator&(const   Query &q1,const Query &q2)
{
    return std::shared_ptr<Query_base> (new AndQuery(q1, q2));
}

class OrQuery : public BinaryQuery
{
    friend Query operator|(const Query &q1, const Query &q2);
    OrQuery(const Query &q1, const Query &a2) : 
        BinaryQuery(q1, a2, "|") {}
    QueryResult eval(const TextQuery &t) const;
};

inline
Query operator|(const   Query &q1,const Query &q2)
{
    return std::shared_ptr<Query_base> (new OrQuery(q1, q2));
}

// '|' 操作
QueryResult
OrQuery::eval(const TextQuery &t) const
{
    QueryResult lqr = lq.eval(t), rqr = rq.eval(t);
    //左QueryResult的行号or上右QueryResult行号
    auto ret_lines = std::make_shared<std::set<int>>(lqr.begin(), lqr.end());
    ret_lines -> insert(rqr.begin(), rqr.end());
    //初始化一个新QueryResult
    return QueryResult(rep(), ret_lines, lqr.get_file());
}

// '&'操作
QueryResult
AndQuery::eval(const TextQuery &t) const
{
    QueryResult lqr = lq.eval(t), rqr = rq.eval(t);
    //一个指向空set的指针
    auto ret_lines = std::make_shared<std::set<int>>();
    ret_lines -> insert(rqr.begin(), rqr.end());
    //使用标准库算法求并集
    set_intersection(lqr.begin(), lqr.end(),
            rqr.begin(), rqr.end(),
            inserter(*ret_lines, ret_lines -> begin()));
    return QueryResult(rep(), ret_lines, lqr.get_file());
}

// '~'操作
QueryResult
NotQuery::eval(const TextQuery &t) const
{
    QueryResult qr = query.eval(t);
    //一个指向空set的指针
    auto ret_lines = std::make_shared<std::set<int>>();
    auto qrb = qr.begin(), qre = qr.end();
    auto sz = qr.get_file() -> size();
    for (auto n = 0; n != sz; ++n)
    {
        if (*qrb != n + 1)
            //如果行号不在查询结果里面，插入到set中
            ret_lines -> insert(n+1);
        else if(qrb != qre)
            ++qrb;
    }
    return QueryResult(rep(), ret_lines, qr.get_file());
}

#endif


//主函数调用
int main () 
{
    ifstream infile("text.txt");
    TextQuery tq(infile);
    auto res = Query("harry") & Query("my") | ~Query("haha");
    cout << res.eval(tq);
/*
    ((harry & my) | ~(haha)): 
    ...
    ...
    ..
*/
}
```

# 模板

16.19

输出容器内容的函数

```
template <typename T>
void my_print(const vector<T> &ve)
{
    //typename指定成员类型
    typename vector<T>::size_type sz = ve.size(), i;
    for(i=0; i != sz; ++i)
    {
        cout << ve[i];
    }
    cout << endl;
}

int main () 
{
    vector<int> vec={1,2,3,4,5,6};
    my_print(vec);
}
```

16.20

使用迭代器的版本

```
template <typename T>
void my_print(const vector<T> &ve)
{
    typename vector<T>::const_iterator bg = ve.begin(), be = ve.end();
    while(bg != be)
    {
        cout << *bg++;
    }
    cout << endl;
}

int main () 
{
    vector<int> vec={1,2,3,4,5,6};
    my_print(vec);
}
```

另外一个版本

```
template <typename T>
ostream  &my_print(ostream &os, const T &ve)
{
    typename T::const_iterator bg = ve.begin(), be = ve.end();
    while(bg != be)
    {
        os << *bg++;
    }
    return os;
}

int main () 
{
    vector<int> vec={1,2,3,4,5,6};
    my_print(cout, vec) << endl;;
}
```
