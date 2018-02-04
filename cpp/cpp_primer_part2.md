---
title: C++ Primer
date: 2016-10-18 19:08:13
tags: [c_plus]
---


# 标准IO库

## IO类

+ IO对象不可复制或赋值
+ 不能储存在容器中
+ 形参和返回类型不能为流类型
+ IO操作通常以引用方式传递和返回流
+ 读写IO对象会改变其状态，所以不能是const引用

```
ofstream &print(ofstream&); //引用
while (print(out2)){..}
```

getline用于逐行读取，保留空格，不保存空格

+ `badbit` 标志着系统级故障
+ `failbit` 标志者可恢复错误
+ `eofbit` 遇到文件结束符

如果达到文件结束位置，eofbit和failbit都会被置位

```
string i;
while(cin>>i) //每次读取一个单词，读取到文件结束符退出循环
{
    cout << i << endl;
}
```

执行clear()之后，复位所有标志位

```
string i,j;
while(cin >> j)
{
    while(cin >> i) //如果这里输入文件结束符
    {
        cout << i << endl;
    }
    cin.clear(); //复位标志位，因此不会退出循环
}
```

+ 输出操作可能不直接输出到屏幕，而是先存入缓冲区
+ 如果程序异常终止，缓冲区不会被刷新
+ 可以手动刷新缓冲区

```
cout << "asd" << endl; //换行、刷新缓冲区
cout << "sad" << flush; //刷新缓冲区
cout << "dsa" << ends; //空格、刷新缓冲区
cout << unitbuf; //后面的所有操作都会立即刷新缓冲区
//...
cout << nounitbuf; //回到正常的缓冲方式
```

关联输入和输出流，任何读取操作都会先刷新输出流

```
cin.tie(nullptr); //cin绑定为空指针
ostream *old_tie = cin.tie(&cout); //cin绑定cout返回一个指向cout的指针
```

## 文件流

### 使用文件流对象

文件流对象的创建和绑定

```
//定义文件对象
ifstream infile;
ofstream outfile;
fstream f;

//捆绑文件
infile.open("in.txt");
outfile.open("out.txt");
f.open("text.txt")

//直接初始化
ifstream infile("in.txt");
fstream f("text.txt")

//检查文件是否成功打开
if (!infile){.... return -1}

//重新捆绑需要先关闭文件
ifstream infile("in.txt");
infile.close(); //关闭文件
infile.open("another.txt");

//清除状态
//一旦文件流发生错误，必须通过clear()方法清除状态，否则会影响后续该对象的使用
infile.clear();

//指定模式打开文件
ofstream outfile("text.txt", ofstream::out | ofstream::trunc);
```

确认成功读取文件之后再操作

```
string s;
fstream f("text.txt");
if (f)
{
    while(getline(f,s))
        cout << s << endl;
    f.close();
}
```

### 文件模式

+ 只可以对ofstream或者fstream对象设置为out
+ 只可以对ifstream或者fstream对象设置为in
+ 只有当设置了out才能设置trunc，默认out + trunc
+ ofstream默认out，ifstream默认in，fstream默认in + out
+ ofstream默认会清空，除非制定app模式 

## 字符串流

在sstream头文件中定义

istringstream 从string读取数据
ostringstream 向string写入数据
stringstream 读取或者写入数据

方法

- `sstream.str()` 返回sstream保存的string的拷贝
- `sstream.str(s)` 将string s拷贝到sstream中

使用istringstream

```
string line,word;
getline(cin, line); //得到一个以空格隔开的字符串
istringstream s(line); //将字符串与s绑定
while(s >> word) //逐个输出空格隔开的单词
{
    cout << word << endl;
}
```

使用ostringstream

```
ostringstream words;
string s;
while(cin >> s) //输入的数据空格隔开，逐个存到words字符串流
{
    words << s << ";";
}
cout << words.str() << endl; //输出所有内容
```

# 顺序容器

## 概述

P292

`vector` 可变大小数组，尾部插入速度快（push_back）
`deque` 双端队列，头尾插入速度快（push_front、push_back）
`list` 双向链表，任何位置插入速度都很快，占用空间大
`forward_list` 单向链表
`array` 固定大小数组，不能添加或者删除元素
`string` 字符串，尾部插入速度快

## 操作

P305

- 同类型容器，同类型元素可以直接用于初始化
- 如果使用迭代器初始化，可以是不同容器类型，不同元素类型（如果能进行转换）
- 同容器同元素可以直接用于赋值
- swap交换除了array以外的顺序容器速度很快，因为只交换了数据结构，元素本身不变
- 只有相同容器类型，相同元素类型可以比较大小
- array定义时需要指定大小

```
//初始化为：1234560000
array<int, 10> a = {1,2,3,4,5,6};
```

assign赋值

赋值操作需要左右两边类型相同，顺序容器（除了array）定义了assign成员，允许不同但是相容的类型赋值

```
vector<const char*> vc={"asdasd", "asdasd"};
list<string> vs;
vs.assign(vc.begin(), vc.end());
```

insert插入元素

第一个参数是一个迭代器，后面可以接收更多参数

```
vector<int> iv;
iv.insert(iv.begin(),10,2); //在开头插入10个2
iv.insert(iv.end(),{1,2,3,4,5,6,7}); //在最后插入1234567
list<int> ls{1,2,3,4,5};
iv.insert(iv.end(), ls.begin(), ls.end()); //使用迭代器插入
```

insert的返回值

C++11 insert返回一个迭代器
但是linux用的g++并不能，折腾好久，坑，好想早点换电脑装VS - -

```
auto tp = iv.insert(iv.begin(),100); //如果是插入一个元素，返回插入的元素的迭代器
cout << *tp << endl;
```

emplace将参数传递给构造函数，使用构造函数构造元素

`emplace_back`、`emplace`、`emplace_front`
对应
`push_back`、`insert`、`push_front`

```
vector<Sales_data> vsd;
vsd.emplace_back("hello", 2, 1);
```

`front`和`back`（除了forward_list）成员函数，返回第一个和最后一个元素值

at（string、vector、deque和array）使用下标返回元素的引用，越界抛出out_of_range异常

```
vector<Sales_data> vsd;
vsd.emplace_back("hello", 2, 1);
auto s = vsd.at(0); //第一个元素的引用
auto &s = vsd.front(); //和上面的等价
```

删除元素
`pop_back`、`erase`（返回被删除元素之后的元素的迭代器）、`pop_front`
对应
`push_back`、`insert`、`push_front`

- 删除deque中的首尾外的元素，会使所有迭代器、引用、指针失效
- 指向vector、string中删除点之后的位置的迭代器、引用、指针失效
- vector不支持push_front

```
vector<Sales_data> vsd;
vsd.insert(vsd.begin(), 10, Sales_data("world",3,2)); //插入是个元素
 //删除一半元素，p指向被删除的最后一个元素的下一个元素
auto p = vsd.erase(vsd.begin(),vsd.begin() + vsd.size()/2);
//删除所有元素，下面两个等价
vsd.erase(vsd.begin(),vsd.end());
vsd.clear()
```

forward_list拥有特殊的访问操作（操作实现方式不同），没有insert、emplace、erase操作（P313）

- `before_begin()`、`cbefore_begin()` 首元素前一个位置的迭代器
- `insert_after` 在某迭代器后插入
- `emplace_after` 在某迭代器后插入，使用构造函数构造
- `erase_after` 删除某迭代器后元素

删除`forward_list<int>`里面的奇数元素 ，关键在于保存要删除元素的前一项的迭代器

```
forward_list<int> fvi{1,2,3,4,5,6,7,1,2,4};
auto fp = fvi.begin(); //第一个元素
auto pre = fvi.before_begin(); //第一个元素的前一个元素
while(fp != fvi.end())
{
    if (*fp%2 != 0)
    {
        fp = fvi.erase_after(pre); /删除值为奇数的元素，fp为下个元素的迭代器
    }
    else
    {
        //两个迭代器都向后移动
        ++fp; 
        ++pre;
    }
}
```

resize用于改变容器大小（array除外）

`c.resize(n)` 调整大小为n
`c.resize(n, t)` 调整大小为n，如果需要添加元素，则添加t

## 容器对象实现

- vector将元素连续存储，每个元素紧挨着前一个元素存储
- 当需要获取新的内存空间时，vector和string的实现通常会分配比新的空间需求大的空间
- `c.capacity()`不重新分配内存空间的话，可以容纳的元素数量
- `c.reserve(n)`分配至少容纳n个元素的空间

## 字符串操作

除了通用的构造方式外，string还支持下标方式构造函数

- `string s(cp, n)` 数组cp的前n个元素（其实就是字符，下同）
- `string s(s2, p)` string s2 下标p开始的元素
- `string s(s2, p, n)` string s2 下标p开始的n个元素

substr 返回一个string（类似切片）

- 如果只有一个参数，获取整个字符串
- 如果有两个参数，第一个参数是开始位置，第二个参数是拷贝的字符数
- 如果第一个参数超过范围，会抛出out_of_range异常
- 如果第二个参数超过范围，substr会调整数值，只拷贝到string末尾

```
string s("hello world!");
cout << s.substr(0, 5) << endl; //输出hello
cout << s.substr(0, 1) << endl; //输出h
cout << s.substr(0, s.size()) << endl; //输出整个字符串
cout << s.substr(0) << endl; //输出整个字符串
```

`insert` 和 `erase` 可以用下标代替迭代器（P323）

`append` 在末尾插入元素（可以不止一个字符）

```
s.append("welcome!");
```

`replace` 是调用erase和insert的一种简写形式

```
string s("hello world!");
s.replace(6, 6, "harry!"); //从第6个开始，删除6个元素，插入"harry!"
cout << s << endl; //输出 hello harry!
```

使用`substr`和`replace`实现字符串的替换

```
void fun(string &s, const string &oldstr, const string &newstr)
{
    string::size_type n = 0;
    while(n != s.size())
    {
        if (s.substr(n, oldstr.size()) == oldstr)
        {
            s.replace(n, oldstr.size(), newstr);
            n += oldstr.size();
        }
        else
            ++n;
    }
}

int main () 
{  
    string s("hello world!");
    fun(s, "world", "harryx");
    cout << s << endl; 输出：hello harryx!
}  
```

string 搜索操作

- 搜索成功返回一个`string:size_type`的下标
- 搜索失败返回`string:npos`，（无符号类型-1，相当于非常大）
- 区分大小写
- 除了一个字符或者字符串或者指针参数，还可以添加一个位置参数，表示搜索开始的位置，默认0
- 或者添加两个参数`(cp, pos, n)`，从字符串中位置pos开始，查找指针cp指向的数组的前n个字符
- `rfind`逆向搜索

```
string s("hello world!");
auto p1 = s.find("o"); //p1等于4
auto p2 = s.find_first_of("abcde"); //p2等于1
auto p3 = s.find_last_of("abcde"); //p3等于10
auto p4 = s.find_first_not_of("abcde"); //p4等于0
auto p5 = s.find_last_not_of("abcde"); //p5等于11
cout << p << endl;
```

`compare`比较

- `s.compare(s2)` s和s2比较
- `s.compare(pos, n, s2)` s中的pos开始的n个字符和s2比较
- `s.compare(pos1, n1, s2, pos2, n2)` s中pos1开始的n1个字符和s2中pos2开始的n2个字符比较
- `s.compare(cp)` s和cp指向的c风格字符串比较
- `s.compare(pos, n, cp) `
- `s.compare(pos1, n1, cp, n2)`

数值转换

to_string(val) 数值转换为字符串
`stoi(s, p, b)` 字符串转换为各种类型，b为基数，默认10
`stol(s, p, b)` p是指针，保存s中第一个非数值字符下标，默认0
`stoul(s, p, b)`
`stoll(s, p, b)`
`stoull(s, p, b)`
`stof(s, p)`
`stod(s, p)`
`stold(s, p)`

# 泛型算法

P367

- 大多数泛型算法都定义在头文件 `algorithm` 中
- 迭代器使得算法不依赖于容器
- 泛型算法不会直接改变容器大小
- 向目的位置迭代器写入数据的泛型算法假定目标容量足够大，否则结果是未定义的
- `fill_n`需要目标容器具有足够大的元素空间`resize`，不仅仅是空间`reserve`

## 只读

`find`算法，前两个参数是迭代器或者指针，第三个参数是一个值
类似的还有`count`算法，计算某个元素出现次数
`fill(v.begin(), v.end(), 0)`写入元素，要注意容器的容量大小
`fill_n(v.begin(), 10, 1)` 写入多个元素

```
//查找vector中的某个元素，返回一个迭代器
vector<int> vi{1,2,3,4,5};
auto res = find(vi.begin(), vi.end(), 3);
cout << *res << endl;

//查找数组中的某个元素，返回一个指针
int ia[] = {1,2,3,4,5,6,7,7};
auto res = find(begin(ia), end(ia), 4);
cout << *res << endl;
```

## 写操作

`back_inserter` 插入迭代器，定义在`iterator`头文件，
接收一个指向容器的引用，返回一个绑定的插入迭代器
插入元素，并且不会出现容器不够大的问题


```
vector<int> vi; //空的容器
auto it = back_inserter(vi); //绑定插入迭代器
*it = 20; //插入一个 20
```

将插入迭代器`back_inserter`和`fill_n`结合使用

```
vector<int> vi;
fill_n(back_inserter(vi), 10, 2 ); //添加 10 个 2
```

copy 拷贝容器的值

```
int a[] = {1,2,3,4,5};
//获取数组大小然后定义一个同样大小的空数组
int b[sizeof(a)/sizeof(*a)];
auto ret = copy(begin(a), end(a), b); //拷贝，ret指向b的end()
```

将插入迭代器`back_inserter`和`copy`结合使用

```
list<int> ii{1,2,3,4,5};
vector<int> vi;
copy(ii.begin(), ii.end(), back_inserter(vi));
```

## 定制操作

可以向泛型算法传递函数参数，使用指定方式操作

```
sort(v.begin(), v.end(), isShorter);使用制定函数isShorter进行排序
```

### lambda表达式

lambda 表达式形式

捕获列表是lambda所在函数中定义的局部变量的列表，通常为空

```
 [捕获列表](参数列表) -> 返回类型 {函数体}
```

如果忽略返回类型，lambda根据代码推断出返回类型
但是必须有捕获列表和函数体
忽略参数列表和返回类型的lambda表达式

```
auto f = [] {return 10;};
cout << f() << endl;
```

传递参数

```
auto f = [](int i, int j) {return i + j;};
cout << f(1, 2) << endl;
```

捕获参数

- 如果需要使用函数中的变量，lambda必须在捕获列表中指明
- 捕获列表只能用于局部非static变量
- lambda可以直接使用局部static变量和所在函数外声明的名字
- 捕获的值在捕获时拷贝
- 可以捕获引用，必须确保在lambda执行时，引用变量是存在的

```
int a = 100, b = 200;
//捕获a和b
auto f = [a, b](int i, int j) {return i + j + a + b;};
```

隐式捕获

在捕获列表中可以只写一个&或者=，代表引用或者值捕获

```
int a = 100, b = 200;
//使用一个=符号，表示值捕获
auto f = [=](int i, int j) {return i + j + a + b;};
```

混合捕获

- 使用混合捕获时，第一个捕获的变量必须是&或者=
- 显示捕获的变量必须使用与隐式捕获不同的方式

```
int a = 100, &b = a;
auto f = [=, &b](int i, int j) {return i + j + a + b;};
```

可变lambda

如果希望改变捕获的变量的值，需要加一个关键字mutable

```
int a = 100;
//如果没有mutable关键字会报错，原捕获的变量是只读的
auto f = [a]() mutable {return ++a;};
a = 0;
auto res = f(); //res=101，a=0，值在创建时拷贝，且不会改变原值
```

指定返回类型

如果有多个return语句，需要指定返回类型

```
auto f = [](int a) -> int 
    {if(a%2==0) return a;else return 0;};
auto j = f(2);
```

### 参数绑定

- `bind`函数可以绑定函数的参数，在头文件`functional`中
- 占位符`_n`在`std::placeholders`命名空间中，n表示第一个参数
- 不能拷贝ostream或者其他引用参数，只能使用`ref`传递`bind(fun, ref(os), _1, 100)`

```
int addnum(int i, int j)
{
    return i+j;
}

int main () 
{  
    //_1是一个占位符，给定参数1，生成新的可调用对象addone
    auto addone = bind(addnum, placeholders::_1, 1);
    cout << addone(1) << endl;
}  
```

## 迭代器

在头文件`iterator`中定义了额外的几个迭代器

-  插入迭代器（insert），绑定到一个容器，执行插入操作
- 流迭代器（stream），绑定到输入输出流，可以遍历所有IO流
- 反向迭代器（reverse），向前移动的迭代器
- 移动迭代器（move），不拷贝元素，而是移动


### 插入迭代器

接受一个容器，生成一个迭代器，能实现向给定容器添加元素

`it = t` 在 it 制定的当前位置插入值 t，根据选择的迭代器的不同调用方法
`*it, ++it, it++` 这些操作虽然存在，不会做任何事，每个操作都返回 It

- `back_inserter`创建一个使用`push_back`的迭代器
- `front_inserter`创建一个使用`push_front`的迭代器
- `inserter` 创建一个使用insert的迭代器，此函数接受第二个参数，是一个指向给定容器的迭代器，元素被插入到给定迭代器之前

只有在容器支持某个操作时才能使用对应迭代器

```
vector<int> vec;
auto it = inserter(vec, vec.end());
*it = 10; //在尾部插入一个10
```

使用front_inserter倒过来拷贝

```
list<int> a{1,2,3,4};
list<int> b;
//b为4,3,2,1
copy(a.begin(), a.end(), front_inserter(b));
```

### 流迭代器

- `istream_iterator`读取输入流
- `ostream_iterator`向输出流写数据

#### istream_iterator操作

- `istream_iterator`可以绑定一个输入流，可以是cin或者一个文件流
- `*in, ++in, in++`读取值、读取下一个值和读取并移动

```
//绑定cin，接受整数输入，如果接受到错误输入，迭代器和尾后迭代器相等
istream_iterator<int> int_it(cin), eof;
//eof为空迭代器，作为尾后迭代器
//使用输入的数字初始化vec
vector<int> vec(int_it, eof);

//同理，使用文件流
ifstream f("text.txt");
istream_iterator<string> str_it(f), eof;
vector<string> vec(str_it, eof);
```

#### ostream_iterator操作

- `ostream_iterator`可以提供第二个参数，一个C风格字符串，每个元素后都会打印这个字符串
- 只要对象有`<<`操作，都可以绑定ostream_iterator
- `out = val`使用`<<`操作，将val写入到out
- `out++,++out,*out`操作存在，但是不做任何事

```
vector<int> vec{1,2,3,4,5};
ostream_iterator<int> out_it(cout, "\n");
//通过copy输出元素
copy(vec.begin(), vec.end(), out_it);//输出12345
```

将一个文件里面的数字按照奇偶性分别写到两个文件中

```
//打开文件
fstream in("text.txt");
fstream out1("out1.txt", ofstream::out);
fstream out2("out2.txt", ofstream::out);
//绑定流
istream_iterator<int> in_it(in), eof;
ostream_iterator<int> out_it1(out1, " ");
ostream_iterator<int> out_it2(out2, " ");
//逐个写入
while(in_it != eof)
{
    if ((*in_it) % 2 == 0)
        out_it1 = *in_it++;
    else
        out_it2 = *in_it++;
}
//关闭文件
in.close();
out1.close();
out2.close();
```

### 反向迭代器

P364

- 除了forward_list，其他容器都支持反向迭代器`reverse_iterator`
- `v.rbegin()`是最后一个元素，`v.rend()`是第一个元素的前一个位置

反向打印vector的元素

```
vector<string> vec{"hello","world","byebye"};
ostream_iterator<string> out_it(cout, ",");
copy(vec.rbegin(),vec.rend(), out_it);//输出：byebye,world,hello
```

# 关联容器

map（保存键-值对）和multimap（关键字可重复出现的map）定义在头文件map中
set（保存关键字）和multiset（关键字可重复出现的set）定义在头文件set中
无序容器定义在头文件unordered_map和unordered_set中

## 使用关联容器

- 关联容器不支持顺序容器中位置相关的操作，比如push_front、push_back
- 初始化map必须提供关键字和值的类型
- map和set关键字必须唯一，multimap和multiset没有此限制

范围for从map中提取的元素是个pair类型对象
pair是个模板类型，保存first、second公有数据成员

```
map<string, size_t> word_count;
++word_count["harryx"];
cout << word_count["harryx"] << endl; //输出1
for (auto w:word_count)
{
    cout << w.first << w.second << endl;//输出harryx1
}
```

set用于检测某个值是否存在

```
set<string> extr{"fuck", "shit"};
string word;
while(cin >> word)
{
    if(extr.find(word) == extr.end())
        cout << word << endl;
}
```

关键字类型的要求

- 有序容器（map、set、multimap、multiset）的关键字必须定义元素的比较方法
- 可以指定自定义的比较方法

```
//自定义比较函数
bool compareSD(const Sales_data &sd1, const Sales_data &sd2)
{
    return sd1.avg_price() < sd2.avg_price();
}
int main () 
{
    //两个自定义类型的实例
    Sales_data sd1{"hello",2,2},sd2{"asd",3,3};
    //第一个关键字是类类型，如果是map，第二个关键字是值，第三个是函数指针
    //第二个关键字是个函数指针（用decltype获取函数类型，加上*号制定为指针）
    //  函数指针相当于bool (*)(const Sales_data &, const Sales_data &)
    //用compareSD初始化bookstore对象，添加元素时用compareSD排序
    set<Sales_data, decltype(compareSD)*> bookstore(compareSD);
    //赋值
    bookstore = {sd1,sd2};
}  
```

## pair类型

- 定义在头文件`utility`中
- 保存两个数据成员，类似容器，是一个用来生成特定类型的模板
- 两个数据成员可以是不同类型
- pair的数据称原是public的，分别命名为first和second

`pair<string, size_t> asam{"harryx", 20};`

操作

- `make_pair(v1, v2)` 返回一个用v1和v2初始化的pair，自动推导类型
-  同类型比较大小，先比较first再比较second

构造一个pair的函数

- 返回vec的最后一个元素作为first，大小作为second
- 如果vec为空，返回一个默认构造值

```
pair<string, int>
process(vector<string> &vec)
{
    if(!vec.empty())
        //较早版本需要显式构造值
        //return pair<string, int>(vec.back(), vec.size());
        return {vec.back(), vec.size()};
    else
        return pair<string, int>();
}

int main () 
{
    vector<string> vec{"asd","dsa","sad"};
    pair<string, int> p;
    p = process(vec);
    cout << p.first << " " << p.second << endl;//输出：sad 3
}  
```

读取string和int 的序列，作为pair保存在vector中

```
int main () 
{
    vector<pair<string, int>> vec;
    istringstream ist;
    string line, word;
    int num;
    //读取空格相隔的string 和int 流
    while(getline(cin, line))
    {
        //字符串流绑定
        ist.str(line);
        //先读取一个字符
        while(ist >> word)
        {
            //尝试读取后面的数字，如果出错就提醒
            if(ist >> num)
            {
                vec.push_back(pair<string, int>(word, num));
                cout << "Ok!Now store " << word << " " << num << endl;
            }
            else
                {
                    cout << word << " bad input" << endl;
                    break;
                }
        }
        //清除错误标记
        ist.clear();
    }
    //输出
    for (auto p:vec)
    {
        cout << p.first << " " << p.second <<  endl;
    }
}  
```

## 关联容器操作

除了一般的容器操作，关联容器定义了额外的类型别名

- `key_type` 容器的关键字的类型
- `mapped_type` 只适用与map，每个关键字所关联的类型
- `value_type` 对于set，与key_type相同，对于map，为pair<const key_type, mapped_type>

```
set<string>::key_type v1;//v1是string 类型
set<string>::value_type v2;//v2同样是string类型
map<string, int>::key_type v3;//v3是string类型
map<string, int>::mapped_type v4;//v4是int类型
map<string, int>::value_type v5;//v5是pair<const string, int>类型
```

### 关联容器的迭代器

- map和set类型都支持begin和end操作，可以通过迭代器遍历容器
- 解引用关联容器的迭代器，会得到一个类型为容器的`value_type`的值的引用
- 可以通过迭代器改变容器的second的值，但不能改变first的值
- set的迭代器是只读的

```
map<string, int> m({"a", 1}, {"b", 2}, {"c", 3});
auto mp = m.begin();
//mp 是pair<const string, int>的引用
cout << mp -> first;
cout << " " << mp -> second;
```

- 关联容器一般不使用泛型算法
- 如果对关联容器使用算法，一般作为源序列或者目的位置
- 关联容器没有push_back()操作，所以不能使用back_inserter

```
vector<int> vec{1,2,3,4,5,6,7,8,9};
multiset<int> mus{1,2,3};
//把vec的元素插入到mus中
copy(vec.begin(), vec.end(), inserter(mus, mus.end()));
//因为是顺序搜索，输出1,1,2,2,3,3,4,5,6,7,8,9
for(auto s:mus)
{
    cout << s;
}
```

### 插入元素

- 关联容器的`insert`成员向容器中添加元素，类似的还有`emplace`
- 向map和set插入已经存在的元素对容器没有影响
- map的元素类型是pair

向set添加元素

```
vector<int> vec{1,2,3};
set<int> se{1,2};
se.insert(vec.begin(), vec.end());//1,2,3
se.insert({4,5,6});//1,2,3,4,5,6
```

向map添加元素

```
map<string, int> m;
m.insert({"asd", 1});
m.insert(make_pair("sdf", 2));
m.insert(pair<string, int>("dfg", 3));
m.insert(map<string, int>::value_type("gfh", 4));
```

- 添加单一元素的insert返回一个pair
- 如果插入成功，first是指向容器中关键字的迭代器，second是一个bool值，如果关键词存在，bool为false
- 如果是允许重复的关联容器，first指向的是新元素的迭代器

### 删除元素

- erase删除一个元素，返回删除的元素的数量，元素不重复的容器返回0或1
- 关联容器的元素按照比较顺序存储

- c.erase(k) 删除关键字k
- c.erase(p) 删除迭代器p指定的元素
- c.erase(b, e) 删除迭代器b，e范围的元素

### 下标操作

- `set`类型不支持下标，因为元素就是关键字
- `map`和`unordered_map`可以使用下标
- `multimap`和`unordered_multimap`不支持下标，因为可能有多个值对应一个元素
- 使用一个不存在的元素作为下标，会添加一个元素到容器中
- 对一个`map`进行下标操作会返回一个`mapped_type`对象，而解引用`map`迭代器会获得`value_type`对象

### 访问元素

- `c.find(k)` 如果存在元素k，返回一个迭代器指向k，否则返回一个迭代器指向c.end()
- `c.count(k)` 如果存在，返回元素的个数，否则返回0
- `c.lower_bound(k)` 返回一个迭代器，指向第一个关键字不小于k的元素
- `c.upper_bound(k)` 返回一个迭代器，指向第一个关键字大于k的元素
- `c.equal_range(k)` 返回一个迭代器pair，表示关键字等于k的元素的范围，若不存在，pair两个成员都为c.end()
- `lower_bound`和`upper_bound`都不适用于无序容器
- 因为是顺序存储，`lower_bound`和`upper_bound`可以用于访问可重复关联容器的某个元素

使用equal_range访问可重复关联容器

```
multimap<string, int> s({"asd", 1}, {"qwe", 2}, {"asd", 2});
for (auto pos = s.equal_range("asd"); pos.first != pos.second; ++pos.first)
    {
        cout << pos.first -> first << " " << pos.first -> second << endl;
    }
```

## 无序容器

P395

- 无序容器不是使用比较运算符来组织元素，而是使用哈希函数（hash function）和关键字类型的==运算符
- 提供了与有序容器相同的操作（find，insert等），遍历的顺序也是无序的
- 无序容器在存储上组织为一组桶

# 动态内存 

`new` 分配内存 `delete` 释放内存

使用动态内存的原因：

+ 程序不知道需要多少对象
+ 程序不知道所需对象的准确类型
+ 程序需要在多个对象间共享数据

## 智能指针

C++11标准提供了两种智能指针来管理动态对象，外加一个伴随类，定义在头文件`memory`中：

+ `shared_ptr`允许多个指针指向同一对象
+ `unique_ptr`“独占”所指对象
+ `weak_ptr`弱引用，指向shared_ptr所管理的对象

P401

`shared_ptr`和`unique_ptr`

### shared_ptr

如果不初始化一个智能指针，会被初始化为一个空指针

```
shared_ptr<string> p;//一个空指针
```

操作
P401

```
p.get(); //返回一个普通指针
p.unique();//是否唯一
p.reset(); //如果p是唯一指向其对象的指针，释放
p.reset(q); //q是内置指针，令p指向q，否则将p置空
p.reset(q, d); //调用自定义方法d释放q
```

初始化后使用

```
int main () 
{
    shared_ptr<string> p = make_shared<string>();//使用make_shared初始化智能指针，指向空string
    if (p && p -> empty()) //如果指针不为空，并且指向一个空string
    {
        *p = "hello"; //解引用赋值
    }
    cout << *p << endl; //输出hello
}  
```

`make_shared`在动态内存中分配一个对象并初始化它
初始化方式类似`emplace`，传入参数用于初始化

```
shared_ptr<string> p = make_shared<string>("hello");
```

使用`auto`简化定义

```
auto p = make_shared<string>("asd");
```

结合使用shared_ptr和new

用于初始化智能指针的指针必须是动态内存，因为智能指针默认使用delete释放对象

如果使用自定义其他类型指针上，需要自定义释放操作

一旦用指向动态内存的指针初始化智能指针，之前的指针就被释放为空悬指针

```
shared_ptr<string> p(new string("hello"));//初始化完，默认调用delete释放动态分配的内存
```

+ `shared_ptr`使用拷贝方式或者初始化方式赋值，都会使引用计数数递增
+ 赋值新值或是被销毁会使引用计数递减，一旦计数器变为0，自动释放管理的对象
+ `shared_ptr`通过**析构函数**完成销毁工作

```
p = q;
auto p(q);
```

自定义释放操作

```
void fun(dest &d)
{
    connection c = connect(d);
    //自定义释放操作end_connection
    shared_ptr<connection> p(c, end_connection);
}
```

其他操作

shared_ptr通过reset更新对象

```
p.reset(new string("a new string"));
```

配合unique操作

虽然使用p.use_count()操作可以获取指针用户数，但是只应用与测试操作
`p.unique()`远比`p.use_count() == 1`效率高。

```
if (!p.unique()) //如果不是唯一用户
    p.reset(new string(*p)); //分配新的拷贝
*p += newval; //改变对象值
```

共享底层数据

```
class StrBlob
{
public:
    typedef vector<string>::size_type size_type;
    //构造函数
    StrBlob():data(make_shared<vector<string>>()){}
    StrBlob(initializer_list<string> ils)://多个字符串参数
        data(make_shared<vector<string>>(ils)) {}
    //成员函数
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

int main () 
{
    StrBlob b1{"a","aa","aaa"};
    StrBlob b2;
    b2 = b1;
    cout << b1.size() << endl; //3
    cout << b2.size() << endl;//3
    b1.push_back("ssss");
    cout << b1.size() << endl;//4
    cout << b2.size() << endl;//4
}  
```

### unique_ptr

初始化`unique_ptr`需要使用直接初始化的方式
`unique_ptr`不支持不同的赋值和拷贝
但是可以拷贝或赋值一个将要被销毁的`unique_ptr`
P418

```
unique_ptr<string> p(new string);//直接初始化

p = q;//不支持赋值
unique_ptr<string> p(q);//不支持拷贝

p = nullptr ; //释放对象，p置空
p.release(); //p放弃对指针的控制权，返回指针，p置空
p.reset(); //释放p
p.reset(q); //如果提供了内置指针（动态内存）q，p指向q，否则释放p
p.reset(nullptr);

unique_ptr<string> p(q.release()); //q控制权转为p

q.release(); //错误做法，并不会释放q，而且丢失了指针
auto q = p.release();//必须手动释放内存：delete(q) 
```

传递unique_ptr参数和返回unique_ptr

```
//返回一个unique_ptr
unique_ptr<int> fun(int p)
{
    return unique_ptr<int> (new int(p));
}
```

unique_ptr管理删除器的方式与shared_ptr不同
所以必须在尖括号中指定函数类型的指针

```
void f(dest &d)
{
    connection c = connect(d);
    unique_ptr<connection, decltype(end_connection) *>
        p (c, end_connection);
}
```

### weak_ptr

指向shared_ptr管理的对象，不会改变share_ptr的引用计数
用shared_ptr初始化

```
weak_ptr<T> w(sp); //sp是shared_ptr
w = p; //p可以是一个shared_ptr或者一个wek_ptr
w.reset(); //w置空
w.use_count(); //与w共享的shared_ptr的数量
w.expired(); //如果use_count()为0返回true
w.lock(); //如果use_count()为true，返回空shared_ptr，否则返回一个指向w对象的shared_ptr
```


安全地访问weak_ptr对象

```
//如果不为空，条件成立
if (shared_ptr<int> p = w.lock())
{
    //pass
}
```

定义一个StrBlob的伴随指针类

```
//先申明
class StrBlobPtr;
class StrBlob
{
friend class StrBlobPtr; //申明为友元
//...
    StrBlobPtr begin();
    StrBlobPtr end();
    //其他和之前一样
    //...
};

class StrBlobPtr
{
public:
    //构造函数
    StrBlobPtr() : curr(0) {}
    StrBlobPtr(StrBlob &a, size_t sz=0):
        wptr(a.data), curr(sz) {} //用shared_ptr初始化弱指针
    //成员函数
    string &deref() const; //解引用
    StrBlobPtr &incr(); //递增
    bool equal( StrBlobPtr b) const //检查索引值
        {return this -> curr == b.curr;}
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

string &StrBlobPtr::deref() const
{
    auto p = check(curr, "dereference past end"); //检查索引
    return (*p) [curr]; //取vector元素
}

StrBlobPtr &StrBlobPtr::incr()
{
    check(curr, "increment past end of StrBlobPtr"); //检查索引
    ++curr; //自增移动
    return *this;
}

int main () 
{
    StrBlob b{"a","aa","aaa"};
    StrBlobPtr bp = b.begin(), be = b.end();
    while(!bp.equal(be))
    {
        cout << bp.deref() << endl; //a aa aaa
        bp.incr();
    }
} 
```

## 直接管理内存

可以使用auto从初始化器推断类型
因为编译器要从初始化器推断类型，只有当括号中仅有单一初始化器才能用auto

```
auto *p1 = new auto(obj); //p1指向与obj类型相同的对象，用obj初始化
auto *p2 = new auto{a, b, c}; //错误，只能有单个初始化器
```

可以动态分配const对象，必须初始化，或者隐式初始化
```
const int *p1 = new const int(100); //分配并初始化const int
const int *p2 = new const string; //分配并默认初始化为空串
```

如果内存耗尽，new表达式会失败，抛出一个类型为`bad_alloc`的异常
可以改变用new的方式，阻止抛出异常
这种形式的new被成为定位new
`bad_alloc`和`nothow`都定义在头文件new中

```
int *p1 = new int; //如果分配失败，抛出std::bad_alloc
int *p2 = new (nothow) int; //如果分配失败，返回空指针
```

`delete`表达式必须指向动态分配的内存，或是一个空指针
释放一块非new分配的内存，或者相同指针释放多次，行为是未定义的
虽然const对象的值不能改变，但是本身是可以被销毁的
直到被释放，动态对象一直存在

```
int *i = new int(1);
delete i; //释放一个动态分配的内存

int *ii = nullptr;
delete ii; //释放一个空指针（虽然不会报错，但并没有什么意义）
```

动态分配vector、添加元素、打印、删除

```
//动态分配一个vector
vector<int> *fun1()
{
    return new vector<int>;
}
//添加元素
void fun2(vector<int> *p)
{
    int temp;
    while(cin>>temp)
    {
        p -> push_back(temp);
    }
}
//打印
void fun3(vector<int> *p)
{
    for(auto i:*p)
    {
        cout << i << endl;
    }
}
int main () 
{
    auto *p = fun1();
    fun2(p);
    fun3(p);
    delete p;//释放空间
}  
```

使用智能指针

```
//返回一个指向vector的智能指针
shared_ptr<vector<int>> fun1()
{
    return make_shared<vector<int>>();
}
//添加元素
void fun2(shared_ptr<vector<int>> p)
{
    int temp;
    while(cin>>temp)
    {
        p -> push_back(temp);
    }
}
//打印
void fun3(shared_ptr<vector<int>> p)
{
    for(auto i:*p)
    {
        cout << i << endl;
    }
}

int main () 
{
    auto p = fun1();
    fun2(p);
    fun3(p);
} 
```

## 动态数组

分配数组会得到一个元素类型的指针，不是数组类型
不能使用begin()和end()操作

```
int *p = new int[get_size()];

int *p = new int[10]; //没有初始化的int
int *p = new int[10](); //10个初始值为0的int
int *p = new int[10]{0,1,2,3,4,5,6,7,8,9};
```

释放

```
delete [] p;
```

### 智能指针和动态数组

标准库提供了可以管理new 分配的数组的unique_ptr版本

```
unique_ptr<int []> up (new int[10]);
up.release(); //自动调用delete[]销毁指针
```

不支持成员访问运算符（点和箭头）
使用下标操作

```
for(size_t i = 0; i != 10; ++i)
    up[i] = i;
for(size_t i = 0; i != 10; ++i)
    cout << up[i] << endl;
```

shared_ptr不支持管理动态数组，除非自定义删除器

```
//使用lambda表达式自定义删除器
shared_ptr<int> sp(new int[10], [](int *p) { delete [] p;});
sp.reset(); //调用删除器释放数组
```

操作

```
    shared_ptr<int> sp(new int[10], [](int *p) { delete [] p;});
    for(size_t i = 0; i != 10; ++i)
        *(sp.get() + i) = i;
    for(size_t i = 0; i != 10; ++i)
        cout << *(sp.get() + i)  << endl;
    sp.reset();
```

### allocator类

定义在头文件`memory`中

指定对象类型，定义allocator对象，然后根据对象类型确定大小
P428

```
allocator<string> alloc; //定义一个allocator对象
auto const p = alloc.allocate(10); //分配10个未初始化的string
alloc.construct(p, args); // args传递给构造函数，构造对象
alloc.destroy(p); //调用析构函数，p必须是一个类型为string*的指针
alloc.deallocate(p, 10); //释放p开始的10个string的内存，在调用前需要先调用destroy
```

## 文本查询程序

```
class QueryResult;
class TextQuery
{
friend class QueryResult;
public:
    TextQuery(std::ifstream &infile);
    QueryResult query(std::string s) const;
private:
   std::shared_ptr<std::vector<std::string>> ifile; //指向存放每行文本的vector
   //这里用int表示行号，实际上应该用非负类型
    std::map<std::string, std::shared_ptr<std::set<int>>> wm; //单词到存放行数的set的映射
};

class QueryResult
{
friend std::ostream &print(std::ostream &os, const QueryResult &qr);
public:
    QueryResult(std::string s, std::shared_ptr<std::set<int>> lp, std::shared_ptr<std::vector<std::string>> fp) : 
        world(s), linep(lp), file(fp){}
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

//打印存放在QueryResult内的内容
std::ostream &print(std::ostream &os, const QueryResult &qr)
{
    os << qr.world << ": \n";
    for (auto num:*qr.linep) //对每个指针解引用然后遍历
    {
        os << num << " "; //行号
        os << *(qr.file -> begin() + num - 1) << std::endl; //用迭代器输出vector的行文本
    }
    return os;
}

int main () 
{
    ifstream infile("text.txt");
    TextQuery tq(infile);
    print(cout, tq.query("harry"));
}
```
