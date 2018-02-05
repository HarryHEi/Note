---
title: javascript-01 语法和数据结构
date: 2016-04-26 08:11:58
tags: [javascript]
---

# JavaScript语法
## 准备工作
JavaScript不需要特殊软件编译执行，只要有Web浏览器就够了。
JavaScript是一种解释型脚本语言，Web浏览器负责解释执行。
打开浏览器，摁F12呼出浏览器的控制台，可以观测运行状况。

JavaScript有两种方式执行。
第一种方式是将JavaScript代码放在head标签中：
```
<!DOCTYPE html >
<html lang="en">
    <head>
        <meta charset="utf-8" />
        <title>JavaScript</title>
        <script type="text/javascript">
                function displayMsg()
                {
                    alert("Hello World!")
                }
        </script>
    </head>
<body>
    <div class="myname">
        Javascript
    </div>
</body>
</html>
```
第二种方式是把JavaScript保存在一个名为.js的独立文件，在文档的head标签部分放一个script标签，并把src指向该文件。
因为脚本默认为JavaScript，所以没必要指定type="text/javascript"属性。
```
<!DOCTYPE html >
<html lang="en">
    <head>
        <meta charset="utf-8" />
        <title>JavaScript</title>
        <script type="text/javascript" src="test.js"></script>
    </head>
<body>
    <div class="myname">
        Javascript
    </div>
</body>
</html>
```
或者把script标签放在HTML文档最后
```
<!DOCTYPE html >
<html lang="en">
    <head>
        <meta charset="utf-8" />
        <title>JavaScript</title>
    </head>
<body>
    <div class="myname">
        Javascript
    </div>
    <script src="test.js"></script>
</body>
</html>
```

## 语法
### 语句
各条语句可以放在不同行就可以分隔他们
```
first
second
```
如果放在同一行，就需要用分号隔开
```
first; second;
```
建议在每条语句后加分号
```
first;
second;
```

### 注释
两个斜杠作为一行的开始，这一行就会被当成注释
```
//这是一条注释
```
使用`"/*    */"`可以多行注释
```
/* 多行注释
注释多行 */
```
或者使用HTML风格注释
```
<!-- 这是HTML风格注释 -->
```
但是为了避免混淆，最好使用双斜杠和`'/**/'`注释
## 条件语句
条件必须放在if后面的圆括号内
```
var a = 100;
if (a>50)
{
    alert(">50");
}
```
if后面可以有一个else语句
```
var a = 20;
if (a>50)
{
    alert(">50");
}
else
{
    alert("<50");
}
```
**比较操作符**，可以是">"、"<"、">="、"<="、"=="、"!="
字符串可以使用"=="号判断是否相同
“==”并不是严格相同，下面这个例子条件语句求值结果是true，相等操作符把空字符串认为与false相同
```
var a = false;
var b = "";
if (a==b)
{
    alert("equal")
}
```
"==="会执行严格的比较，下面条件语句执行结果为False，同样的，严格的不等使用"!=="
```
var a = false;
var b = "";
if (a===b)
{
    alert("equal")
}
```
**逻辑操作符**包括，与"&&"、或"||"、非"!"

## 循环语句
while循环
```
var a = 10;
while (a>0)
{
    alert(a);
    a --;
}
```
do...while循环
```
var a = 10;
do
{
    alert(a)
    a++;
}
while(a<20)
```
for循环
```
for (var count=1; count<10; count++)
{
    alert(count);
}
```

## 函数
定义函数
```
function shout (num)
{
    for (var count=0; count<num; count++)
    {
        alert(count);
    }
}
shout(10)
```

# 数据结构
2016年5月10日编辑，参考《学习javascript数据结构与算法》一书

## 变量
声明变量
```
var name;
```
赋值
```
name = "javascript";
```
JavaScript允许直接对变量赋值而不需要声明
```
name = "javascript";
alert(name);
```
或者
```
var name="javascript";
```
JavaScript变量名可以是字母数字下划线，不过数字不能是第一个字符
```
var my_name = "javascript";
```
另一种方式是使用驼峰格式(camel case)
```
var myName = "javascript";
```

## 操作符
除了支持：`“+ - * / % ++ -- > < >= <= == = += -= *= /= %= && || ! & | ~ ^ >> <<”`
还支持："typeof delete(删除对象属性)"

## 数据类型
### 变量
JavaScript不需要进行类型声明
下面两句都是合法的
```
var age = "thirty three";
age = 33;
```
字符换必须包含在引号里面，单引号双引号都可以，如果字符串已经有引号，就得用另外一个引号
```
var name = "i don't know";
```
或者使用转义字符
```
var name = 'i don\'t know';
```
如果给一个变量赋值，数值可以是整数，也可以是浮点数
```
var num = -2.33333;
```
还有一种数据类型是布尔类型，他只有两个可选值true或者false，注意这不是字符串，不能用引号
```
var select = true;
```

### 数组
创建一个数组
```
var list = Array(4);
```
也可以不指定长度
```
var list = Array();
```
给数组赋值
```
var list = Array();
list[0] = 100;
list[1] = "RMB";
alert(list[0]);
alert(list);
```
创建数组的时候赋值
```
var list = Array(100, "RMB", 200, "dollors");
```
也可以使用方括号
```
var list = [100, "RMB"];
```
或者
```
var list = [];
list[0] = 100;
list[3] = 200;
alert(list);
```
可以用字符串代替数字值，这样的数组称为**关联数组**
实际上创建的是一个Array对象属性，本身的内容并没有被填充，下例就是给list添加了name、year两个属性
```
var list = [];
list["name"] = "harry";
list["year"] = "100"
alert(list); //返回空
alert(list["name"]); //返回harry
```
添加元素
在数组末尾添加元素，可以直接赋值
```
var list = [1,2,3,4];
list[list.length] = 5;
```
或者使用push方法
```
var list = [1,2,3,4];
list.push(5);
```
在首部添加，使用unshift方法
```
list.unshift(-2,-1,0);
```
删除元素
删除最后一个元素，使用pop方法
```
list.pop();
```
删除数组的第一个元素，使用shift方法
```
list.shift();
```
删除指定位置的元素，使用splice方法
```
list.splice(3,2); //删除索引3开始的2个元素
```
数组中插入元素
```
list.splice(5,0,1,2,3); //第二个参数是删除元素的个数，这里是0，第三个参数往后就是要添加的元素
```
二维和多维数组
数组元素可以是数组
```
var list = [1,2,3,[1,2,3,[1,2,3]]];
alert(list);
```
构成一个二维数组
```
car list=[]
list[0]=[1,2,3,4]
list[1]=[5,6,7,8]
```

### 迭代器
every迭代器会迭代每个元素，直到返回false
```
> lis=[1,2,3,4,5]
> lis.every(function(x){return x>0})
true
> lis.every(function(x){return x>2})
false
```
some迭代器会迭代每个元素，直到返回true
forEach会迭代整个数组
```
> lis.forEach(function(x){console.log(x)})
1
2
3
4
5
undefined
```
map会遍历并返回数组的元素，返回的仍然是数组
```
> mymap=lis.map(function(x){return x})
[ 1, 2, 3, 4, 5 ]
> lis=[1,2,3,4,5]
> lis.some(function(x){return x>2})
true
> lis.some(function(x){return x>6})
false
```

### 排序
数组排序使用sort方法，排序就相当于比较两个数的值
```
> lis.sort()
[ 1, 2, 3, 4, 5 ]
> lis.sort(function(a,b){return a-b})
[ 1, 2, 3, 4, 5 ]
> lis.sort(function(a,b){return b-a})
[ 5, 4, 3, 2, 1 ]
```

### 搜索
indexOf方法返回元素的索引值，返回最先找到的那个
lastIndexOf返回最后找到的元素的索引
```
> lis[1,1,2,2,3,3,4,5]
3
> lis.indexOf(3)
4
> lis.lastIndexOf(3)
5
```

### 数组转化为字符串
toString输出所有元素为字符串
join用一个分隔符把元素分开
```
> lis
[ 1, 1, 2, 2, 3, 3, 4, 4, 7, 6, 4, 4 ]
> lis.toString()
'1,1,2,2,3,3,4,4,7,6,4,4'

> lis.join('-')
'1-1-2-2-3-3-4-4-7-6-4-4'
> lis.join('--')
'1--1--2--2--3--3--4--4--7--6--4--4'
```

### 对象
创建一个**对象**
```
var student = Object();
student.name="harry";
student.year = 100;
alert(student.name);
```
或者使用花括号
```
var student = {
    name:"harry",
    year:100
};
alert(student.name);
```
对象也可以放在数组中
```
var student = {
    name:"harry",
    year:100
};
var list = [];
list[0] = student;
alert(list[0].name);
```

## 栈
栈遵循后进先出
```
function Stack() 
{
    var items = [];
    this.push = function(element)
    { //添加元素到栈顶
        items.push(element);
    };
    this.pop = function()
    { //移除栈顶元素
        return items.pop();
    };
    this.peek = function()
    { //返回栈顶元素
        return items[items.length-1];
    };
    this.isEmpty = function()
    { //空则返回true
        return items.length == 0;
    };
    this.size = function()
    { //返回元素个数
        return items.length;
    };
    this.clear = function()
    { //移除所有元素
        items = [];
    };
    this.print = function()
    { //打印所有内容
        console.log(items.toString());
    };
}
```

## 队列
遵循先进先出
```
function Queue() 
{
    var items = [];
    this.enqueue = function(element)
    { //队列尾添加元素
        items.push(element);
    };
    this.dequeue = function()
    { //队列头移除元素
        return items.shift();
    };
    this.front = function()
    { //返回队列头元素
        return items[0];
    };
    this.isEmpty = function()
    { //空返回true
        return items.length == 0;
    };
    this.clear = function()
    { //清空队列
        items = [];
    };
    this.size = function()
    { //返回长度
        return items.length;
    };
    this.print = function()
    { //打印队列
        console.log(items.toString());
    };
}
```

## 变量作用域
如果在使用局部变量时没有声明，有可能会改变全局变量的值，下例最后输出30，全局变量值改变了
```
function shout (num)
{
    for (var count=0; count<num; count++)
    {
        res ++;
    }
}
var res=20;
shout(10);
alert(res);
```
正确做法是，先声明，下例输出20
```
function shout (num)
{
    var res=0
    for (var count=0; count<num; count++)
    {
        res ++;
    }
}
var res=20;
shout(10);
alert(res);
```

## 对象
对象(Object)是非常重要的数据类型，可以通过两种形式访问对象里的数据：属性(property)和方法(method)

+ 属性是隶属于某个特定对象的变量
+ 方法是某个特定对象才能调用的函数

在JavaScript中，属性和方法都使用"."来访问
如果对象不存在，会返回undefined
```
var a = ['A', 'B', 'C'];
a.name="hello"; //和a["name"]="hello"含义相同
```
可以自由地添加删除属性
```
var a = {
    name:"harry",
    second_name:"x"
}
delete a.second_name
alert(a.second_name) //返回undefined
```
可以为给定对象创建一个新实例
```
var xiaom = new Person;
```
内建对象
数组就是一种内建对象
```
var list = new Array
list.length //返回数组长度
```
类似的还有Math和Date对象
```
var num = 7.561
var num = Math.round(num) //十进制转化为整数
var current_date = new Date()
var date = current_date.getDay() //星期几
```

## Map和Set
Map是一组键值对结构
```
var names = ["a","b","c"];
var ages = [10,20,30];
var m =new Map([["a",10], ["b",20], ["c",30]]);
alert(m.get("a")); //返回10
```
添加键值，一个键只能对应一个值，多次赋值会覆盖掉
```
m.set("d",40)
```
Set和Map类似，也是key的集合，但是不存储value，重复的key会被自动过滤
```
var s =new Set([1,2,3,4,1,2]);
>>> s //返回Set [ 1, 2, 3, 4 ]
```
添加和删除键
```
s.add(5)
s.delete(3) //返回true
```

## iterable
遍历Array可以采用下标循环，遍历Map和Set就无法使用下标。为了统一集合类型，ES6标准引入了新的iterable类型，Array、Map和Set都属于iterable类型。
for...of循环遍历
```
var a = ['A', 'B', 'C'];
var s = new Set(['A', 'B', 'C']);
var m = new Map([[1, 'x'], [2, 'y'], [3, 'z']]);
for (var x of a)  // 遍历Array
{
    alert(x);
}
for (var x of s)  // 遍历Set
{
    alert(x);
}
for (var x of m)  // 遍历Map
{
    alert(x[0] + ':' + x[1]);
}
```
当给Array添加属性之后，for...in遍历会出问题
```
var a = ['A', 'B', 'C'];
a.name="array";
for (var x in a)
{
    alert(x); //返回0，1，2，name
}
```
for...of遍历修复了这个问题
```
var a = ['A', 'B', 'C'];
a.name="array";
for (var x of a)
{
    alert(x); //返回A，B，C
}
```
更好的方式是使用iterable内置的forEach方法，接收一个函数，每次迭代会自动回调该函数
```
var a = ['A', 'B', 'C'];
a.forEach(function (element, index, array) {
    // element: 指向当前元素的值
    // index: 指向当前索引
    // array: 指向Array对象本身
    alert(element);
});
```
Set没有索引，回调函数的前两个参数都是元素本身，即element的值和index的值相同
Map的回调函数参数依次是value、key和map本身
