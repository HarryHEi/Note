---
title: C++ Primer
date: 2016-08-31 21:20:27
tags: [c_plus]
---

# 开发环境
## 编译工具
`Linux`下使用`GCC`编译器对`C++`程序进行编译
```
g++ hello.cc -o hello
g++ hello.cc students.cc -o hello //编译多个文件

g++ -std=c++11 hello.cc -o hello // c++11 标准
```
`makefile`的编写

## 编辑器
使用sublime编辑器，ctrl+B可以直接编译

# 基础
## 输入输出（iostream库）
```
#include <iostream>
int main()
{
    int v1,v2;
    std::cout << "Enter two numbers" << std::endl;
    std::cin >> v1 >> v2;
    std::cout <<  "the sum is " << v1 + v2 << std::endl;
    return 0;
}
```
下面两种流操作是等价的
```
std::cout << "Enter two numbers" << std::endl;

std::cout << "Enter two numbers";
std::cout << std::endl;
```
`std::`表示命名空间`std`，`::`是作用域操作符（scope operator）
或者使用using声明命名空间，`using std::string`

## 变量、基本类型
c++11 标准支持使用花括号初始化
```
int val{0};
```
变量声明和定义
```
extern int a; //声明，不定义
```
类类型通过**构造函数**（constructor）初始化

`const`定义的变量值不会被意外修改，如果定义为全局变量，不能被其他文件访问

`const`必须初始化，一旦创建，值无法修改

多个文件共享const变量，需要添加`extern`关键字

在`****.cc`文件中定义
```
extern const int aaa=10;
```
在`****.h`头文件中声明
```
extern const int aaa;
```

`&`引用是对象（变量）的别名，这里的`i`是变量（对象），`ii`是引用
```
char i='a';
char &ii = i;
```

+ 引用不是对象，没有实际地址
+ 引用一旦定义，无法再赋值

`const`引用可以绑定到相关类型对象或右值（字面值等）上，称为常量引用
```
int &a = 1; //错误，不能为非const引用绑定常量（字面值）
const int &a = 1; //正确，可以用const引用绑定常量
double i=2.1;	//i => 2.1
int &ii = i; //错误，不能使用非const绑定临时量
const int &ii = i;	//ii => 2 这里的  2 只是一个临时量，所以必须使用const
i++;	//i => 3.1 , ii => 2
```
可以通过其他途径修改原变量的值
```
int i = 42;
int &r1 = i;
const int &r2 = i;  //这里绑定了 i ，不过不能通过r2修改 i 的值
r1 = 0; //通过 r1 修改了值
cout << "r1: " << r1 << " r2: " << r2 << endl; //输出: r1: 0 r2: 0
```

## 指针
定义一个指针
```
string s("hello world");
string *sp = &s; 
```
定义一个空指针
```
int  *p = 0;
int *p = nullptr; //C++11标准
```
`void*`指针可以存放任意对象地址
```
int i = 20;
void *p = &i;
```

+ 迭代器用于访问容器内元素，指针指向单个对象，保存另一个对象地址
+ 指向const的指针才能指向const对象，也可以指向非const对象，但是不能修改对象值，指针本身的值可以修改
+ const指针指针值不能修改

```
const int s = 1;
const int *sp = &s;//指向const类型的指针，指针值可以修改（重新指向一个const int 变量
                                        //但是不能修改原变量（不管原变量是不是const）
int *const csp = &s;//错误，const指针应该指向非const int
const int *const  pa = &a;//指向const int 的 const 指针
```

+ 顶层`const`：指针本身是常量
+ 底层`const`：指针所指对象是常量

常量表达式`constexpr`

常量表达式可以定义为`constexpr`
```
constexpr int i = 20;
constexpr int a = i + 2;
constexpr int *ip = nullptr;//指向整数的常量指针
```

## 类型
使用typedef定义类型别名
```
typedef double wow, *p; //wow 等同于 double ，p等同于 double *
wow val = 1.1; //double val = 1.1;
p p1 = &val; // double *p1 = &val;
```
C++11使用别名声明
```
using wow = double;
wow val = 1.1;
```

**注意**
`pstring`是指向`char`的指针，数据类型是指针
`const`修饰之后，`const pstring`是指向`char`的`const`指针（常数指针）
`const pstring cstr`相当于`char *const cstr`

```
typedef char *pstring; //pstring 相当于 char*
char a='0',b='1'; 
const pstring cstr = &a; //这并不等于const char *cstr ，这样修饰类型变成char
    //cstr是一个指向char的const指针，char *const cstr
cstr = &b; //错误，因为cstr是const指针，不能重新赋值
```

`auto`让编译器推算变量类型

```
int a{1},b{2};
auto c = a + b, d = a * b; //如果有多个语句所有变量基本数据必须同一类型
                                                //（编译器会适当改变结果）
```

编译器会适当修改结果

```
int a = 1, &r = a;
auto b = r; //b是一个int 类型，并不是引用
a += 1;
cout << a << r << b << endl;//a r都是2，b为1
```

`auto`会忽略顶层`const`，留下底层`const`

```
int a = 0;
const int i = 1;
auto j = i; // j 为int类型
j += 1; //合法
i += 1; //不合法，i 为const int 类型
auto k = &a; // k是 int 类型指针
auto m = &i; //m 是 const int 类型指针对const 的取址是底层const
```

明确指出`const auto`，修饰为顶层`const`

```
const int i = 1; //是不是cosnt 都一样
const auto j = i; //j 为 const auto类型
```

如果给const 变量绑定引用，初始值顶层const保留，保留的const就是底层const

```
const int ci = 1; //const int 类型
auto &a = ci; //a为 const int 类型 的引用
a += 1; //非法

int i = 1; //int类型
const int ci = i; //const int 类型
auto &j = i,*p = &ci ; //非法，j 为 int 类型引用，p为const int 类型指针，不一致
const auto &j = i,*p = &ci ; //合法，均为const int 类型
```

`decltype`返回操作数的数据类型

```
int fun(int i)
{
    cout << "ok" << endl;
    return i+1;
}
int main () 
{
    decltype(fun(1)) i = 10; //编译器并不会真的调用fun()函数
}
```

`decltype`如果表达式是变量，会返回包括顶层const的类型

```
const int a=0;
decltype(a) i = 10; //const int类型
decltype(a) &j = a; //const int 引用
const int &b = a; 
decltype(b) i = a; //const int 引用
```

`decltype`如果表达式不是变量：

+ 如果是表达式，为结果的类型
+ 如果是解引用，为引用类型

```
int a = 0, *p = &a, &r = a;
decltype(r + 0) b = 1; //b 是int类型
decltype(*p) c = a; // c是int引用（int&）
```

注意：
变量加上括号就会被理解为表达式，变量作为表达式是赋值，会得到引用类型

```
int a = 0;
decltype(a) c = a; //c是int类型
decltype((a)) d = a; //d是int引用(int&)
```

## 标准库
### `string`对象

在头文件`string`中

**注意：string对象和字符串字面值不是一回事**

初始化
```
string s1{"hello,world"};
string s2 = "hello world";
string s3 = s2;
string s4 = (10, 'h'); // "hhhhhhhhhhh"
```

string对象操作

|指令|操作|
|----------------------|---------------------------------|
|`getline(cin,s)`|读取一行内容给`s`直到换行符（不包括换行符）|
|`s.empty()`	|如果s为空返回true|
|`s.size()`	|返回长度，`string::size_type`**（一个无符号整型）**|
|`s[n]`		|返回字符，下标为`string::size_type`类型，但可以使用任何整型|
|`s1 + s2`	|拼接，至少有一个是string对象(左结合)|
|`s1 = s2`	|替换`s1`内容|
|`s1 == s2`	|判断相等 (> < >= <= !=大小写敏感)| 

string对象中字符判断（cctype头文件中），一般返回int非0值或0

`isalnum`		字母或数字
`isalpha`		字母
`isdigit`		数字
`isgraph`		不是空格，可以打印
等

读取字符

```
while (cin >> s){ \*   ...   *\ } //每次读取一个单词（不包括空格）
                //然后进入循环体，读取到结束符（ctrl D）退出循环
getline(cin, s); //读取一行直到换行符（读取但是不保存换行符）
```

比较大小

```
string a{"abc"};
string b{"aac"};
string c{"ab"};
//a > b > c
```

使用下标遍历修改字符串

```
string s{"hello,world!"};
for (decltype(s.size()) index = 0; index != s.size(); ++index) //decltype 获取类型
    if (isalpha(s[index]))
        s[index] = toupper(s[index])s
cout << s << endl;
```

C++11 提供新的语句：范围for语句，遍历一个序列

```
for (元素:序列)
    ...
```

使用范围for遍历一个string

```
string s{"hello world"};
for (auto c: s) //使用auto让编译器自己决定变量类型
    cout << c << endl;
```

使用引用遍历并修改字符串

```
string s{"hello,world!"};
for (auto &c: s) //定义原字符的引用
    if (isalpha(c))
        c = toupper(c); //改为大写
cout << s << endl;
```

### `vector`对象
`vector`对象在`vector`头文件中，一个容器只能保存同一种对象（类型），甚至可以是vector对象

定义时指定类型

```
std::vector<int> ivec{1,2,3,4};
std::vector<int> ivec = {1,2,3,4};
std::vector<int> ivec = (10);//10个元素，初始化为0
std::vector<int> ivec(10, -1);//{-1, -1,-1, -1, -1, -1, -1, -1, -1, -1};
std::vector<int> iven(1,2,3,4);//错误

std::vector<string> sven{10};//因为10并不能赋值给string所以，编译器尝试默认初始化
        //为10个默认初始化元素
```

使用push_back添加元素，不能使用下标添加元素

```
vector<int> ivec;
for (int i = 0;i < 100; ++i)
{
    ivec.push_back(i);
}
```

范围for循环访问，**不能在范围for循环添加元素**，因为范围for语句预存了end()的值

```
for (string s: svec)
    cout << s << endl;
```

`vector`对象操作

|指令|操作|
|----------------------|---------------------------------|
|`v.empty()`		|如果为空返回true|
|`v.size()`		|返回长度，`vector<...>::size_type`类型，**不能省略`<...>`**|
|`v.push_back(t)`	|在v尾部添加元素t|
|`v[n]`			|返回元素|
|`v1 = v2`		|v1元素替换|
|`v1 == v2`		|相等返回true(> < >= <= !=)以字典顺序比较|


### 迭代器
所有标准库容器都可以使用迭代器

string也可以使用迭代器
```
string s("hello world");
auto si = s.begin(), se = s.end();
while(si != se)
{
    if (isalpha(*si))
        *si = toupper(*si);
    ++si;
}
cout << s << endl;
```

迭代器有`iterator`和`const_iterator`（只读）类型

可以使用auto让编译器自己判断类型

`iterator`类型

```
vector<int>::iterator iter = v.begin(); //初始化为起始位置

auto b = v.begin(), e = v.end(); //起始元素位置和最后一个元素后一个位置（尾迭代器）

++iter; //迭代器自增移动
--iter;//自减移动

std::cout << *iter; //使用解引用操作符（`*`操作符）访问元素

```

循环迭代到最后一个元素

```
for(vector<int>::iterator iter = v.begin();iter != v.end();++iter)
{
    std::cout << *iter;
}
```

`const_iterator`类型

```
const vector<int> iv{1,2,3,4,5};//当定义为常量容器时
auto v = iv.begin(), ve = iv.end();//下面两句等价
vector<int>:: const_iterator v = iv.begin(), ve = iv.end();
```

`cbigin`和`cend`获取`const_iterator`

```
auto v = iv.begin(), ve = iv.end();
```

元素访问

```
vector<string> iv{"hello","world","","ok"};
auto v = iv.begin(), ve = iv.end();
(*v).empty();使用字符串的empty方法检查是否为空
v -> empty();和上面等价
```

### `bitset`对象

`bitset`对象在头文件`bitset`中

定义`bitset`对象时指定长度： `bitset<32> b;`

`bitset`对象操作

|指令|操作|
|----------------------|---------------------------------|
|`b.any`		|是否存在1|
|`b.none`		|不存在1|
|`b.count`		|1的个数，size_t类型（cstddef头文件中）|
|`b.size`		|二进制位个数|
|`b[pos]`		|pos位的二进制数|
|`b.test(pos)`		|pos位是否为1|
|`b.set`		|所有位置1|
|`b.set(pos)`		|pos位置1|
|`b.reset`		|所有位置0|
|`b.reset(pos)`	|pos位置0|
|`b.flip`		|取反|
|`b.flip(pos)`		|pos位取反|
|`b.to_ulong()`	|返回一个unsigned long 值|
|`os << b`		|输出到流|

# 数组
## 数组定义和初始化

+ 非const变量以及只有在运行后才能知道值的const变量不能用于定义数组维度
+ 使用字符串初始化数组，默认在最后添加一个空字符 `'\0'`（C风格字符串）
+ 如果数组元素类型为类，自动调用类默认构造函数初始化
+ 定义数组必须指定数组类型，不能使用auto由初始值推导，数组元素为对象，所以不存在引用的数组
+ 数组的索引是`size_t`类型
+ 指针（指向数组元素）相减是`ptrdiff_t`类型（带符号）

```
int *p[10];//指针数组
int (*p)[10];//数组指针
int &p[10];//错误，没有引用的数组
int (&p)[10];//数组的引用
int *(&p)[10];//指针数组的引用
```

支持范围for

```
int a[10]={1,2,3,4,5,6,7,8,9,10};
for (int i: a)
{
    cout << i << endl;
}
```

C++11标准引入begin和end函数

```
int a[10]={1,2,3,4,5,6,7,8,9,10};
int *p = begin(a);
int *pe = end(a);
while(p != pe)
{
    cout << *p << endl;
    ++p;
}
```

可以使用数组初始化`vector`，反过来不行

```
int a[] = {1,2,3,4,5}
vector<int> iv(begin(a), end(a))
```

## C风格字符串标准库函数

`cstring`是`string.h`头文件的C++版本

可以使用C风格字符串来初始化string对象，反过来不行

可以使用string的`c_string()`方法来初始化，不能保证一直有效，最好拷贝一份

```
string as("hello world");
const char *str = as.c_str();//指针类型是const char*
cout << strlen(str) << endl;
```

C风格字符串

```
int main()
{
    const char *a = "hello";
    char b[] = "world";
    char d[40]; //要足够大，否则会出错
    strcpy(d, b); //b拷贝给d
    strcat(d,b); //b附加到d后面
    std::cout  <<  strlen(a) << std::endl;
    std::cout  <<  strlen(b) << std::endl;
}
```
## 动态数组

C中使用`malloc`和`free`分配储存空间
C++中使用`new`和`delete`

`new`表达式返回指向新分配数组的第一个元素的指针
```
int *pia = new int[10];
```

如果数组元素具有类类型，使用默认构造函数进行初始化，或者使用括号
```
string *psa = new string[10];
int *pib = new int[10]();
```

数组长度可以在程序运行时动态决定，并且，长度0也是合法的

使用`delete`释放内存，**注意**方括号不能缺少
```
delete [] pia;
```

# 表达式

`sizeof()`

+ 返回类型所占字节数
+ 返回指针本身所占空间大小
+ 返回解引用指针所指对象大小，指针不需有效
+ 返回整个数组所占空间大小
+ 不会返回string和vector的元素所占空间，返回类型固定部分大小

```
int a[] = {1,2,3,4,5,6,7};
int *p = a;
sizeof(a);//4*7=28
sizeof(p);//28
sizeof(*p);//4
```

## 逗号运算符

从左往右计算，丢弃左值，保留最右侧表达式的值

## 类型转换

强制类型转换

```
cast-name<type>(expression); 
```

`cast-name`是`static_cast`、`dynamic_cast`、`const_cast`和`reinterpret_cast`中的一种

`static_cast`
只要不包含底层`const`，就可以使用

```
int i{10}, j{19};
double slope = static_cast<double>(j) / i;
cout << slope << endl;//输出1.9

//用于找回存在与void*指针中的值
double d = 10;
void *p = &d;
double *dp = static_cast<double *>(p);
cout << *dp << endl;//输出10
```

`const_cast`只能改变对象的底层const

```
const int ci = 1;
const int *cp = &ci;
int *p = const_cast<int *>(cp);
*p += 1;
cout << ci << *cp << *p << endl;//输出：1 2 2
```

`reinterpret_cast`为运算对象的位模式提供较低层次上的重新解释

使用`reinterpret_cast`非常危险，本质上依赖机器

# 异常处理语句

`throw` 语句用于抛出异常，可以使用`try`捕获

```
#include <iostream>  
using namespace std;
int main () 
{  
    try  
    {  
        if (1)
            throw 1;
    }  
    catch(int i)  
    {  
        cout << i  << endl;  
    }  
    return 0;
}  
```

使用标准异常类

```
#include <iostream>  
#include <exception>  
using namespace std;
int main () 
{  
    try  
    {  
        int *myarray = new int[10000000000];
    }  
    catch(exception& e)  
    {  
        cout << e.what() << endl;   //输出 std::bad_alloc
    }  
    return 0;
}  
```

`stdexcept`头文件

```
#include <iostream>  
#include <stdexcept>
using namespace std;
int main () 
{  
    int val(10);
    if (val == 10)
    {
        throw runtime_error("test err");//中断程序运行
    }
    cout << "ok" << endl;
    return 0;
} 
```

使用try catch捕获

```
int main () 
{  
    int val(10);
    try
    {
        if (val == 10)
        {
            throw runtime_error("test err");
        }
    }catch (runtime_error err)
    {
        cout << err.what();
    }
    cout << "ok" << endl; //后面继续运行
    return 0;
} 
```

# 函数
## 参数传递
### 引用形参

引用形参，相当于将形参与实参绑定

```
#include <iostream>  
using namespace std;

void swap(int &v1,int &v2)
{
    int temp = v2;
    v2 = v1;
    v1 = temp;
}

int main () 
{  
    int a(1),b(2);
    swap(a,b);
    cout << a << b << endl;
}  
```

使用const引用可以避免复制，提高效率

```
bool longer(const string &s1,const string &s2)
{
    return s1.size() < s2.size();
}
```

如果不用修改参数，就把形参定义为const，否则传入const参数会出现编译错误

```
#include <iostream>
#include <cstring> 
using namespace std;

bool longer(string &v1,string &v2) //非const
{
    return v1.size() < v2.size();
}

int main () 
{  
    if(longer("asdasd","asdasda")) //编译出错
    {
        //
    }
}  
```

**指针引用**

`int *&v1`是指针引用形参，&v1理解为指针，所以需要在前面添加`*`符号

从右往左读符号，最靠近变量名的有最直接影响

下例只交换了两个指针的地址，而a、b的内容并没有交换

```
#include <iostream>  
using namespace std;

void swap(int *&v1,int *&v2)
{
    int *temp = v2;
    v2 = v1;
    v1 = temp;
}

int main () 
{  
    int a(1),b(2);
    int *pa = &a;
    int *pb = &b;
    cout << *pa << *pb << endl; //1 2
    cout << a << b << endl; //1 2
    swap(pa,pb);
    cout << *pa << *pb << endl; // 2 1
    cout << a << b << endl; // 1 2
} 
```

### 迭代器形参

vector等容器把迭代器作为参数传递，下例输出6个8

```
#include <iostream>  
#include <vector>
using namespace std;

void print(
    vector<int>::const_iterator beg,
    vector<int>::const_iterator end
)
{
    while (beg != end)
    {
        cout << *beg++ << ' '; //++符具有更高优先级，== *(beg++)
    }
}

int main () 
{  
    vector<int> v(6,8);
    print(v.begin(),v.end());
    return 0;
} 
```
### 使用标准库规范

传递指向数组首元素和尾后元素的指针

```
void print(const int *begin, const int *end)
{
    while(begin != end)
    {
        cout << *begin++ << endl;
    }
}

int main () 
{  
    int a[] = {0,1,2,3,4,5,6,7,8,9};
    print (begin(a), end(a));
} 
```

### 数组引用形参

除了传递数组指针外，还可以使用引用传递数组`int (&array)[]`
引用后，array和a地址相同，即为同一个东西

```
#include <iostream>  
#include <vector>
using namespace std;

void print(int (&array)[10])
{
    for (auto i: array)
        cout << i << endl;
}

int main () 
{  
    int a[] = {0,1,2,3,4,5,6,7,8,9};
    print (a);
} 
```

**注：**
    `int *array[10]` 是十个指针组成的数组 :指针数组
    `int (*array)[10]` 是十个元素的数组的指针:数组指针

### 传递多维数组

传递多维数组时需要传递两个维度的参数

```
//这两个是等价的
void print(int (*array)[10], int rowsize){ ... }
void print(int array[][10], int rowsize){ ... }
```

### main函数传递参数

`argv`是一个数组，0是文件名，往后是输入的参数，`argc`表示数组中字符串的数量

```
int main (int argc, char *argv[]) 
{  
    string s1, s2;

    s1 = argv[1];
    s2 = argv[2];

    cout << s1 + s2 << endl;
} 
```
输入
```
./hello hello world
```
输出
```
helloworld
```

### 含有可变形参的函数

如果所有参数类型相同，可以使用`initializer_list`标准库类型，在同名头文件中  (198页)

和vector不一样的是，该对象中的元素永远是常量值，无法改变元素值

```
void err_message(initializer_list<string> il)
{
    for (auto beg = il.begin(); beg != il.end(); ++beg)
    {
        cout << *beg << " ";
    }
    cout << endl;
}

int main () 
{  
    err_message({"test", "err"}); //放在一对花括号内
    err_message({"func", "test", "ok"});
} 
```

## 函数返回值

函数不能返回局部变量的引用或者指针，因为临时对象的空间已经释放
函数可以返回一个引用作为左值，注意函数返回类型是引用

```
#include <iostream>  
using namespace std;

char &get_val(string &str,string::size_type ix)
{
    return str[ix];
}

int main () 
{  
    string s("asdasd");
    get_val(s,0) = '-';
    cout << s << endl;//输出：-sdasd
} 
```

### 列表初始化返回值

C++11新标准规定，函数可以返回花括号包围的值的列表

```
vector<string> process()
{
    return {"test", "return", "vector"};
}

int main () 
{  
    vector<string> sv = process();
    for (auto s: sv)
    {
        cout << s << endl; //输出：test return vector
    }
} 
```

### 递归

注意 `char*`类型到`int`类型的转换

```
int fun(int val)
{
    if (val != 1)
    {
        return fun(val - 1) * val;
    }
    return 1;
}

int main (int argc, char *argv[]) 
{  
    int num = atoi(argv[1]); //类型转换
    cout << fun(num) << endl; //输入：./hello 4 输出：24
} 
```

### 返回数组的指针

声明一个数组指针函数 (205页)

```
int array[10];//含有10个元素的数组
int (*p)[10] = &array ;//数组的指针
int (*func(int i))[10]; //返回数组指针的函数
```

+ `func(int i)`函数接受一个整数实参
+ `(*func(int i))`可以对函数调用结果进行解引用
+ `(*func(int i))[10]`解引用后得到的是大小为10的数组
+ `int (*func(int i))[10]`数组的元素类型为`int`

举例

```
int array[10];

int (*func(int i))[10] //接受一个参数，将数组的元素全部赋值为该值
{
    int (*p)[10] = &array;
    for (auto &elem: array)
    {
        elem = i;
    }
    return p;
}

int main () 
{  
    int (*pp)[10] = func(2); //接受参数2，返回一个数组的指针
    for (auto elem: *pp)
    {
        cout << elem << endl;//输出10个2
    }
} 
```

使用尾置返回类型方式
C++11新标准使用**尾置返回类型**简化

```
//func接受一个int类型实参，返回一个指针，这个指针指向含有10个整数元素的数组
auto func(int i) -> int(*)[10];
```

函数改成这样

```
auto func(int i) -> int(*)[10]
{
    int (*p)[10] = &array;
    for (auto &elem: array)
    {
        elem = i;
    }
    return p;
}
```

如果已经知道具体的数组，使用`decltype`获取类型

```
int array[10];

decltype(array) *func(int i) //decltype返回的是个数组，所以还需要添加*符号
{
    int (*p)[10] = &array;
    for (auto &elem: array)
    {
        elem = i;
    }
    return p;
}
```

## 重载函数

重载函数是相同的函数名不同参数的函数，（main函数不能重载）
重载函数在参数数量或者参数类型上有所不同
不允许 "除了返回类型其他都相同" 的情况

```
#include <iostream>  
using namespace std;

void hi(const int i) //const int 参数
{
    cout << i << endl;
}

int hi(const string str)//const string 参数
{
    cout << str << endl;
    return str.size();
}

int main () 
{  
    hi(hi("hello world"));
} 
```

### 重载和const形参

顶层`const`不影响传入函数的对象
引用和指针通过判断是否是常量区分

```
lookup(Phone);
lookup(Phone const);//错误，重复声明了lookup(Phone)

lookup(Phone *);
lookup(Phone *const);//错误，重复声明了lookup(Phone *)

lookup(Phone &);//Phone的引用
lookup(const Phone &);//正确，常量引用

lookup(Phone *);//指向Phone的指针
lookup(const Phone *);//正确，指向常量的指针
```

`const_cast`在重载函数的情景中最有用

```
const string &shortstring(const string &s1, const string &s2) //接收常量
{
    return s1.size() <= s2.size() ? s1: s2;
}

//重载
string &shortstring(string &s1, string &s2) //接受string引用
{
    auto &r = shortstring(const_cast<const string&>(s1),  
        const_cast<const string&>(s2)); //使用const_cast强制转换为常量
    return const_cast<string&>(r);
}

int main () 
{  
    string s1{"hello"};
    string s2{"world!"};
    cout << shortstring(s1,s2) << endl;//传入string引用
    cout << shortstring("justfor","test") << endl;//传入常量
} 
```

### 内联函数

内联函数可以避免函数调用的开销

```
//使用inline关键词定义
inline 
const string &shortstring(const string &s1, const string &s2) 
{
    return s1.size() <= s2.size() ? s1: s2;
}

int main () 
{  
    cout << shortstring("justfor","test") << endl;//传入常量
} 
```

相当于

```
cout <<  s1.size() <= s2.size() ? s1: s2 << endl;
```

### `constexpr`函数

`constexpr`是指能用于常量表达式的函数，遵守以下规定：

+ 函数的返回类型和形参必须是字面值，或者可以是但不一定是另一个常量表达式函数
+ 有且只有一个return语句
+ 常量表达式函数和内联函数一般定义在头文件中

```
constexpr int new_sz() {return 2;}
constexpr int scale(int t) {return new_sz() * t;}

int main () 
{  
    int array[scale(2)]; //相当于int array[4]
    cout << scale(2) << endl;//输出4
    int i  = 2;
    int a2[scale(i)] = {1,2,3,4};//错误，scale(i)不是常量表达式
} 
```

### 调试

`assert`预处理宏，定义在`cassert`头文件中
如果`expr`为假，输出信息，终止程序执行，如果`expr`为真，什么也不做

```
assert(expr)
```

预处理宏由预处理器管理，无需使用`std::`和`using`声明

```
int main () 
{  
    int i = 0;
    cin >> i;
    assert(i > 5);//如果输入3，输出：Assertion `i > 5' failed
    cout << i << endl;
} 
```

`assert`依赖于`NDEBUG`预处理变量的状态，如果定义了`NDEBUG`，`assert`什么也不做
编译器提供命令选项定义这个预处理变量，相当于在文件一开始写`#define NDEBUG`

```
g++ hello.cc -o hello -D NDEBUG
```

也可以自己写调试代码

```
int main () 
{  
    int i = 0;
    cin >> i;
    #ifndef NDEBUG
        cout << "i is: " << i << endl;
    #endif
    cout << i << endl;
} 
```

`C++`编译器定义了一些局部变量

```
__func__ 当前函数名，是const char的一个静态数组
__FILE__ 存放文件名的字符串字面值
__LINE__ 存放当前行号的整型字面值
__TIME__ 存放文件编译时间的字符串字面值
__DATE__ 存放文件编译日期的字符串字面值
```

### 函数匹配

精准匹配（相同、数组->指针、函数->指针、删除或添加顶层const） > const转换 > 类型提升 > 标准转换（算术转换、指针转换） > 类类型转换（219）

指针值（const char*）可以隐式转换为bool类型，为标准转换
指针值（const char*）到sting为类类型转换
所以第二个匹配度更高

```
void find(const string &s1, const string &s2, bool b = "true")
{
    cout << "1"  << endl;
}
void find(const string &s, bool b = "true")
{
    cout << "2" << endl;
}

int main () 
{  
    find("hello","world");//输出：2
} 
```

### 函数指针

函数的类型由返回类型和形参类型共同决定，声明一个指向函数的指针，需要指定返回类型和形参类型

```
bool (*pf) (const string &, const string &);//没有初始化的函数指针
bool shortstring(const string &, const string &);//一个函数
pf = shortstring;//pf指向shortstring函数
pf = &shortstring;//取址符是可选的
pf("hello", "world");//调用
(*pf)("hello", "world");//等价的
```

和数组类型相似
虽然不能定义函数类型的形参，但是可以定义指向函数的指针的形参

```
//第三个参数是形参类型，自动转换成函数指针（精准匹配）
void whichshort(const string &s1, const string &s2, bool (*pf)(const string &, const string &))
//第三个参数是个函数指针，等价
void whichshort(const string &s1, const string &s2, bool (*pf)(const string &, const string &))
//函数作为参数，自动转换
whichshort(s1, s2, shortstring)
```

使用decltype简写

```
typedef decltype(shortstring) Fun;//函数类型
typedef decltype(shortstring) *FunP;//函数指针
void whichshort(const string &s1, const string &s2, Fun);//下面俩和之前等价
void whichshort(const string &s1, const string &s2, FunP);
```

虽然不能返回函数，但是可以返回函数指针，然而要写出返回类型
```
//使用类型别名
using F = int(int*, int);//函数类型
using PF = int (*)(int*, int);//函数指针
//定义
PF f1(int);//返回函数指针的函数
F *f1(int);//等价的
//相当于
//返回一个接受int* 和int参数返回int的函数的指针，接受一个int参数的函数
int (*f1(int)) (int*, int);
```

使用尾置返回类型的方式

```
auto f1(int) -> int(*)(int*, int);
```

和返回数组指针类似

```
auto func(int i) -> int(*)[10];
```

使用decltype，注意`*`符号不能少
```
decltype(fun) *f1(int);
```

# 类
## 自定义数据类型
定义一个自定义数据类型`Sales_data`，这并不是抽象数据类型，需要编写操作
```
struct Sales_data
{
    string  name;
    unsigned unit = 0;
    double revenue  = 0.0;
};
```

### 定义成员函数

- 成员函数必须声明在内部，定义可以在外部或者内部，定义在内部的函数是隐式的（inline）
- 非成员函数必须定义和声明在外部
- 即使数据成员定义在成员函数之后，也可以使用数据，编译器首先编译成员的声明，然后才是成员函数体
- 外部定义的成员函数除了与声明必须匹配外，必须包含类名
- `combine()`函数类似于`+=`，返回的是`this`对象，即调用这个函数的对象，像这样调用`some.combine(another);`

```
struct Sales_data
{
    //成员函数
    string getname() const {return name;}
    Sales_data& combine(const Sales_data&);
    double avg_price() const;
    //数据成员
    string  name;
    unsigned unit = 0;
    double revenue  = 0.0;
};
//非成员函数
Sales_data add(const Sales_data&, const Sales_data&);
ostream &print(ostream&, const Sales_data);
istream &read(istream&, const Sales_data);
//外部定义成员函数
Sales_data& Sales_data::combine(const Sales_data &sd)
{
    unit += sd.unit; //将 sd 的成员加到 this 的对象上
    revenue += sd.revenue;
    return *this; //返回调用这个函数的对象
}
double Sales_data::avg_price() const
{
    if (unit)
        return revenue/unit;
    else
        return 0;
}
```

- 调用`some.getname()`就相当于`getname(&some)`
- 成员函数通过一个隐式的`this`参数来访问对象

```
string getname() const {return name;}
//上面这句也可以这么定义，但一般不这么做
string getname() const {return this -> name;}
```

- `getname()`函数后有一个`const`关键字，作用是修改隐式`this`指针的类型
- 默认情况下，`this`的类型是指向类类型的常量指针`Sales_data *const`
- `getname()`函数不会修改`this`所指对象，因此应该把this设置为指向常量指针
- 即`const Sales_data *const`
- `const`关键字位置是紧跟在参数列表后

### 定义非成员函数

- 一般把一些辅助函数，比如add、read和print等定义为非成员函数
- 定义非成员函数也和其他函数一样，通常把函数的声明和定义分开
- 因为IO类型不能被复制，所以用引用作为参数
- `print()`不负责换行，一般来说，执行输出的函数应该尽量减少对格式的控制，由用户决定
- 调用方式：`read(cin, some)`、`print(cout, some) << endl`

```
istream &read(istream &is, Sales_data &sd)
{
    double price = 0;
    is >> sd.name >> sd.unit >> price;
    sd.revenue = price * sd.unit;
    return is;
}

ostream &print(ostream &os, const Sales_data &sd)
{
    os << sd.getname() << " " << sd.unit << " "
        << sd.revenue << " " << sd.avg_price();
    return os;
}
```

- `add()`函数中，使用`sd1`来初始化`sum`
- 然后调用`combine()`函数把`sd2`数据成员加到`sum`中

```
Sales_data add(const Sales_data &sd1, const Sales_data &sd2)
{
    Sales_data sum = sd1; //把sd1的数据成员拷贝给sum
    sum.combine(sd2); //相当于 +=
    return sum;
}
```

### 构造函数

- 构造函数的任务是初始化类对象的数据成员，类创建时执行
- 构造函数的名字和类名相同，没有返回类型，不能声明为`const`
- 如果类定义没有提供初始值，编译器隐式定义一个默认构造函数
    - 如果类内成员有初始值，使用其初始化
    - 否则，默认初始化该成员
- 如果一个构造函数为所有形参提供了默认参数，也相当于是个默认构造函数
- 构造函数类似重载函数，类可以包含多个构造函数
    - `= default`是C++新标准，表示要求编译器生成默认构造函数
    - 如果定义了其他构造函数而不指明`=default`，使用默认构造函数将出错
    - 括号内为参数（可能为空），花括号内是函数体（可能为空）
    - 之间的是构造函数初始值列表
- 如果是const、引用或未提供默认构造函数的类类型，必须通过构造函数初始化列表而非赋值提供初值
- 在构造函数前加关键字`explicit`可以阻止隐式转换
- 函数前加`~`是析构函数，对象删除时自动调用
```
struct Sales_data
{
    //构造函数
    Sales_data() = default; //C++11标准，表示要求编译器生成默认构造函数
    Sales_data(const string &s) : name(s) {} //使用构造函数初始值列表
    Sales_data(const string &s, unsigned n, double p) :
        name(s), unit(n), revenue(p*n) {}
    Sales_data (istream &); //内部声明，外部定义
    //成员函数
    string getname() const {return name;}
    Sales_data& combine(const Sales_data&);
    double avg_price() const;
    //数据成员
    string  name;
    unsigned unit = 0;
    double revenue  = 0.0;
};
//外部定义构造函数，初始值列表为空，函数体进行初始化
Sales_data::Sales_data(istream &is)
{
    read(is, *this);
}
```

使用构造函数初始化
```
Sales_data sd1; //默认构造函数，初始化为：'' 0 0
Sales_data sd2("hello"); //初始化为：'hello' 0 0
Sales_data sd3("hello", 2, 2); //初始化为：'hello' 2 4
Sales_data sd4(cin); //根据输入初始化

print(cout, sd3) << endl; //输出：hello 2 4 2ll
```

### 委托构造函数

```
//非委托构造函数
Sales_data(const std::string &s, unsigned n, double p) :
    name(s), unit(n), revenue(p*n) {}
//委托构造函数
Sales_data() : Sales_data("", 0, 0) {}
Sales_data(std::string s) : Sales_data(s, 0, 0) {}
Sales_data (std::istream &is) : Sales_data() {read(is, *this);}
```

### 项目结构

- 类结构体定义和非成员函数声明放在`sales_data.h`中
- 注意，不要在头文件中使用`using namespace std;`

```
#ifndef SALES_DATA_H
#define SALES_DATA_H 

#include <iostream>
#include <string>

struct Sales_data
{
    //构造函数
    Sales_data() = default; //C++11标准，表示要求编译器生成默认构造函数
    Sales_data(const std::string &s) : name(s) {} //使用构造函数初始值列表
    Sales_data(const std::string &s, unsigned n, double p) :
        name(s), unit(n), revenue(p*n) {}
    Sales_data (std::istream &); //内部声明，外部定义
    //成员函数
    std::string getname() const {return name;}
    Sales_data& combine(const Sales_data&);
    double avg_price() const;
    //数据成员
    std::string  name;
    unsigned unit = 0;
    double revenue  = 0.0;
};
//非成员函数声明
std::istream &read(std::istream &is, Sales_data &sd);
std::ostream &print(std::ostream &os, const Sales_data &sd);
Sales_data add(const Sales_data &sd1, const Sales_data &sd2);

#endif
```

- 外部定义的成员函数和非成员函数放在`sales_data.cc`中

```
#include "sales_data.h"

using namespace std;

//外部定义成员函数
Sales_data& Sales_data::combine(const Sales_data &sd)
{
    unit += sd.unit;
    revenue += sd.revenue;
    return *this;
}
double Sales_data::avg_price() const
{
    if (unit)
        return revenue/unit;
    else
        return 0;
}

//定义非成员函数
istream &read(istream &is, Sales_data &sd)
{
    double price = 0;
    is >> sd.name >> sd.unit >> price;
    sd.revenue = price * sd.unit;
    return is;
}

ostream &print(ostream &os, const Sales_data &sd)
{
    os << sd.getname() << " " << sd.unit << " "
        << sd.revenue << " " << sd.avg_price();
    return os;
}

Sales_data add(const Sales_data &sd1, const Sales_data &sd2)
{
    Sales_data sum = sd1;
    sum.combine(sd2);
    return sum;
}

//外部定义构造函数
Sales_data::Sales_data(istream &is)
{
    read(is, *this);
}
```

- 在主文件使用include导入

```
#include <iostream>
#include <string>
#include "sales_data.h"

using namespace std;

int main () 
{  
    Sales_data sd(cin);
    print(cout, sd) << endl;
} 
```

## 访问控制和封装

- 自定义数据类型没有强制要求使用接口，仍可访问对象内部
- 使用访问说明符加强类的封装性
    - 定义在`public`说明符后的成员可以在整个程序内被访问
    - 定义在`private`说明符后的成员只能被类的成员函数访问，即隐藏了类的实现细节

当使用`class`关键字，并且添加了`private`说明符之后，部分函数（非成员函数）不能正常编译了
`class`和`struct`关键字唯一区别是默认访问权限变化了
```
class Sales_data
{
public:
    //构造函数
    Sales_data() = default; //C++11标准，表示要求编译器生成默认构造函数
    Sales_data(const std::string &s) : name(s) {} //使用构造函数初始值列表
    Sales_data(const std::string &s, unsigned n, double p) :
        name(s), unit(n), revenue(p*n) {}
    Sales_data (std::istream &); //内部声明，外部定义
    //成员函数
    std::string getname() const {return name;}
    Sales_data& combine(const Sales_data&);
    double avg_price() const;
private:
    //数据成员
    std::string  name;
    unsigned unit = 0;
    double revenue  = 0.0;
};
```

类类型

- 类可以在定义之前声明，比如：`class Salse_data;`
- 不过注意在声明之后、定义之前，编译器并不清楚类的具体内容（不完全类型），期间不能使用其成员
- 成员函数体在整个类成员可见之后才被处理
- 类型名的定义通常出现在类的开始处，确保使用该类型的成员都出现在类型名定义之后

友元

- 类也可以让其他类或者函数访问它的非公有成员，可以把函数声明为友元
- 声明为友元，需要添加一条以关键字`friend`开头的函数声明
- 友元声明可以出现在类内的任意位置，但最好在类定义开头或者结尾集中声明
- 友元的声明仅仅指定了访问权限，所以还需要再次声明以供用户调用
- 指定一个类作为友元，那么那个类就可以访问此类的所有成员
- `All_date`就是`Sales_data`的友元，`All_data`内成员函数可以调用`Sales_data`所有成员
- 注意友元没有传递性，`Salses_data`的友元的友元并不能访问其内部成员
- 如果只将某个类的某个成员函数声明为友元，需要注意步骤
    - 先定义`All_data`类，声明但不定义`clear()`函数，如果用到`Sales_data`类，需要先申明`class Salse_data`
    - 然后定义`Salse_data`类，声明`clear()`友元，注意在定义之前不能访问类内成员
    - 最后再定义`clear()`函数
-如果将重载函数声明为友元，需要将每个函数依次声明

正常的声明友元
```
class Sales_data
{
//非成员函数友元声明
friend std::istream &read(std::istream &is, Sales_data &sd);
friend std::ostream &print(std::ostream &os, const Sales_data &sd);
friend Sales_data add(const Sales_data &sd1, const Sales_data &sd2);
friend class All_data;//友元类
public:
    //构造函数
    Sales_data() = default; //C++11标准，表示要求编译器生成默认构造函数
    Sales_data(const std::string &s) : name(s) {} //使用构造函数初始值列表
    Sales_data(const std::string &s, unsigned n, double p) :
        name(s), unit(n), revenue(p*n) {}
    Sales_data (std::istream &); //内部声明，外部定义
    //成员函数
    std::string getname() const {return name;}
    Sales_data& combine(const Sales_data&);
    double avg_price() const;
private:
    //数据成员
    std::string  name;
    unsigned unit = 0;
    double revenue  = 0.0;
};
//非成员函数声明
std::istream &read(std::istream &is, Sales_data &sd);
std::ostream &print(std::ostream &os, const Sales_data &sd);
Sales_data add(const Sales_data &sd1, const Sales_data &sd2);

class All_data
{
public:
    using SaleIndex = std::vector<Sales_data>::size_type; //容器index类型
    void clear(SaleIndex i);
private:
    std::vector<Sales_data> sales_datas{Sales_data("hello",2,2)}; //存放Salse_data的容器
};

inline
void All_data::clear(SaleIndex i) //清空容器内某个Salse_date类型的内容
{
    Sales_data &sd = sales_datas[i];
    sd.name = std::string("");
    sd.unit = 0;
    sd.revenue = 0;
    sd.num = 0;
}
```

只将类的某个成员函数声明为友元
```
class Sales_data; //因为用到`Sales_data`，需要先声明

class All_data
{
public:
    using SaleIndex = std::vector<Sales_data>::size_type;
    void clear(SaleIndex i); //声明 clear 函数
    Sales_data & getSale(SaleIndex i) {Sales_data &sd = sales_datas[i]; return sd;}
private:
    std::vector<Sales_data> sales_datas; //因为`Sales_data`还没有定义，并不能初始化
};

class Sales_data //定义 Sales_data 类
{
friend class All_data;
/* ... */
};
/* ... */

inline
void All_data::clear(SaleIndex i) //定义 clear 函数
{
    /* ... */
}
```

- 定义在内部的成员函数默认为内联函数（inline），也可以指定外部定义的成员函数为（inline）
- 注意外部定义的inline成员函数应该与类定义放在同一个文件中

```
inline
Sales_data& Sales_data::combine(const Sales_data &sd)
{
    unit += sd.unit;
    revenue += sd.revenue;
    return *this;
}
inline
double Sales_data::avg_price() const
{
    if (unit)
        return revenue/unit;
    else
        return 0;
}
```

- 成员函数可以被重载，这里重载了`numadd()`函数
- 即使是const成员函数，也可以修改数据成员，只要加上`mutable`关键字（mutable data member 可变数据成员）
- 当调用`sd.numadd()`时，数据成员`num`自增 1，同样的`sd.numadd(3)`自增3

```
class Sales_data
{
public:
    //构造函数
    /* ... */
    //成员函数
    void numadd() const {++num;}
    void numadd(int i) const {num += i;}
private:
    //数据成员
    /* ... */
    mutable int num = 0;
};
```

- 返回`*this`的成员函数可以实现链式调用：比如`myScreen.move(4,0).set('*')`
- 如果其中有一个成员设置了`const`关键字，链式调用时会报错，因为返回的是常量引用，后续不能进行写入操作
- 通过重载可以解决这种问题
- `do_display()`函数无论是不是常量引用，都隐式转换成常量引用，然后输出
- 使用`do_dispay()`避免多处重复同样的代码，功能明显，方便调试，隐式被声明为内联函数，运行时没有额外开销

```
class Sales_data
{
/* ... */
public:
    //构造函数
    /* ... */
    Sales_data &addnum() {++num; return *this;}
    Sales_data &addnum(int i) {num += i; return *this;}
    Sales_data &display(std::ostream &os){do_display(os); return *this;} //返回一个非常量引用
    const Sales_data &display(std::ostream &os) const {do_display(os); return *this;} //返回一个常量引用
private:
    //数据成员
    /* ... */
    void do_display(std::ostream &os) const {os << name << " num " << num;} //提供给内部成员函数调用
};
```

链式调用
```
Sales_data sd("hello", 2, 2);
sd.display(cout).addnum(10).display(cout); //这里是非常量引用
```

### 类的静态成员

- 静态成员可以是`public`或者`private`的
- 静态成员函数不包含`this`指针，不能声明成`const`
- 静态成员不与任何对象绑定在一起，使用域运算符直接访问`Sales_data::count`
- 可以在类的内部或者外部定义静态成员函数
- 外部定义静态成员时，不能重复`static`关键字，该关键字只能出现在内部声明语句中
- 静态数据成员不由类的构造函数初始化，必须在类的外部定义和初始化每个静态成员，`int Sales_data::count = 0;`
- 静态数据成员的定义最好和非内联函数放在一个文件中，`sales_data.cc`
- 通常情况下，类的静态成员不应该在类内初始化，如果是字面值常量类型的`constexpr`，可以提供`const`初始值
- 例如：`static constexpr int period = 30`，用于类内定义维度`double array[period]`

静态成员的声明和定义
```
class Sales_data
{
public:
    static int getcount() {return count;} //没有const关键字
    static void addcount() {count += 1;}
private:
    static int count; //内部声明，外部定义
    static constexpr int period = 20; //字面值常量类型的constexpr，内部定义并初始化
    double array[period]; //使用静态成员作为维度
};
```

外部定义静态成员，文件`Salse_data.cc`
```
#include "sales_data.h"

int Sales_data::count = 0;

//定义非成员函数
/* ... */
```
