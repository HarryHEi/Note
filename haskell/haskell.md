---
title: Haskell 函数式编程 (一)
date: 2016-05-24 10:19:01
tags: [haskell]
---


# 简述
Haskell 是一种函数式编程语言

参考:
[Haskell 趣学指南](http://fleurer-lee.com/lyah/)
[Real World Haskell](http://cnhaskell.com/)
[阮一峰的网络日志](http://www.ruanyifeng.com/blog/2012/04/functional_programming.html)

# 函数式编程
函数式编程主要思想是把运算过程尽量写成一系列嵌套的函数调用

1. **函数是第一公民**
函数和其他数据类型相同，可以赋值或者作为参数

2. **只用表达式，不用语句**
每一步都是远算，并且都有返回值，避免不必要的读写(I/O)

3. **没有副作用**
函数只会返回一个值，不会改变其他外部变量

4. **不修改状态**
不修改变量，使用参数保存状态

5. **引用透明**
函数的运行不依赖外部变量，只依赖于输入参数

# Haskell环境
Haskell 使用的编译器是ghc，命令行输入ghci 进入交互界面

自定义提示符
```
Prelude> :set prompt "gchi> "
gchi> 
```
使用`:l`从外部导入一个文件
```
gchi>  :l test.hs
```
使用`:m`从外部导入一个模块
```
Prelude> :m +Data.Ratio
Prelude Data.Ratio> 
```
设置每次返回结果类型
```
Prelude> :set +t
Prelude> 2
2
it :: Integer
```

# 基础
## 数值
### 数组和字符串
Haskell 使用空格调用函数
```
gchi> succ 1
2
```

在交互界面使用let和脚本中定义是等价的
```
gchi> let add x y = x+y
gchi> add 1 2
3
```

字符串或者数组的拼接
```
gchi> let a=[1,2,3,4,5]
gchi> a++a
[1,2,3,4,5,1,2,3,4,5]
gchi> 1:a
[1,1,2,3,4,5]
gchi> null a
False
gchi> null []
True
```

字符串转义
putStrLn用于打印一个字符串
```
Prelude> putStrLn"lalala\n\tlalala"
lalala
	lalala
```

长度，比较符是比较长度
```
gchi> length a
5
gchi> a > [1,3]
False
```

读取值
```
gchi> a
[1,2,3,4,5]
gchi> head a
1
gchi> tail a
[2,3,4,5]
gchi> last a
5
gchi> init a
[1,2,3,4]
gchi> take 3 a
[1,2,3]
ghci>drop 2 a
[3,4,5]
gchi> maximum a
5
gchi> minimum a
1
```

其他操作
```
gchi> a
[1,2,3,4,5]
gchi> reverse a
[5,4,3,2,1]
gchi> sum a
15
```

建立数组
```
gchi> [1..10]
[1,2,3,4,5,6,7,8,9,10]
gchi> [1,3..10]
[1,3,5,7,9]
gchi> cycle[1,2,3]
1,2,3死循环
gchi> repeat 10
10 死循环
```

### 集合
可以添加任意多个限制条件
```
gchi> [x*2 | x <- [1..10]]
[2,4,6,8,10,12,14,16,18,20]

gchi> [x*2 | x <- [1..10],x*2>10]
[12,14,16,18,20]

Prelude> [x | x <- [50..100], x `elem` [60..70]]
[60,61,62,63,64,65,66,67,68,69,70]

Prelude> [x | x <- [50..100], x `mod` 10 ==0]
[50,60,70,80,90,100]

Prelude> [x | x <- [50..60], odd x]
[51,53,55,57,59]

Prelude> [x*y | x <- [1..5],y <- [3..7]]
[3,4,5,6,7,6,8,10,12,14,9,12,15,18,21,12,16,20,24,28,15,20,25,30,35]
```

### 元组
元组(tuple)内容可以是不同类型
```
Prelude> (1,'a',"asd",[1,2])
(1,'a',"asd",[1,2])
```

#### 序对
序对就是有两个元素的元组

使用fst和snd获得序对元素
```
Prelude> fst (1,"dfgh")
1
Prelude> snd (1,"dfgh")
"dfgh"
```

### zip
打包两个list，来生成一组序对
```
Prelude> zip [1,2,3,4] ['a','b','c','d']
[(1,'a'),(2,'b'),(3,'c'),(4,'d')]
```
## 数值类型
数值类型首字母大写，变量首字母小写
整数：Integer
分数：Ratio Integer
分数使用操作符 % 构建

### Haskell类型特点
#### 强类型
拒绝执行无意义的表达式，类型错误会在编译时显示

#### 静态类型
编译器在编译时便直到所有值和表达式的类型

#### 类型推到
编译器自动推断出所有表达式类型

### 查看类型
```
Prelude> :t "asdads"
"asdads" :: [Char]

Prelude> :t (123,"asd",[1,2,3])
(123,"asd",[1,2,3]) :: (Num t, Num t1) => (t, [Char], [t1])
```

### 函数类型
```
Prelude> lines "lalala\nlalala\nlalala\tlalala\n"
["lalala","lalala","lalala\tlalala"]
it :: [String]
Prelude> :t lines
lines :: String -> [String]
```
这里的`->`可以读作映射

类型a是个类型变量，指的是可以取任意类型

fst将两个类型作为参数，返回第一个项的类型
```
Prelude> :t fst
fst :: (a, b) -> a
```
不纯函数（受环境变量影响的函数）的类型以IO开头
```
Prelude> :t readFile
readFile :: FilePath -> IO String
```

在定义函数前声明类型
```
testCase :: Bool -> Bool 
testCase value =
    case value of
        True -> False
        False -> True
```

### 类型类

只有属于响应的类型类才能使用相应的方法

Eq：判断相等
Ord：比较大小
Show：可用字符串表示
Read：字符串转化为一个类型
Enum：可枚举
Bounded：成员有上下限
Num：数字
Intergral：数字
Floating：浮点

```
*Main> read "12" :: Int
12
*Main> read "12" :: Float
12.0
```

使用逗号可以给函数添加多个约束

```
checkReverse :: (Eq a,Num a) => [a] -> Bool
checkReverse xs = if null xs
    then True
    else if xs == (reverse xs)
        then True
        else False
```

### 常见类型
Int：表示整数，有上限
Integer：无边界整数
Float ：浮点数
Double：双精度浮点数
Bool：布尔值
Char：一个字符
Haskell有个特殊的类型：`()`表示一个空元组

## 语法
### 变量
Haskell里变量不能多次赋值

文件：test.hs
```
x=20
x=10
```
导入出错
```
Prelude> :t readFile
readFile :: FilePath -> IO String
Prelude> :l test.hs
[1 of 1] Compiling Main             ( test.hs, interpreted )

test.hs:2:1:
    Multiple declarations of `x'
    Declared at: test.hs:1:1
                 test.hs:2:1
Failed, modules loaded: none.
```

### 条件
在Haskell中，代码缩进预示着一个定义的延续，非常重要

不同分支的类型必须相同，否则会出错
同样的，也不能省略分支

自定义一个drop函数，null用于检查列表是否为空
```
myDrop n xs = if n<=0 || null xs
    then xs
    else myDrop (n-1) (tail xs)

*Main> myDrop 2 [1,2,3,4,5,6]
[3,4,5,6]
```
用Python实现
```
def myDrop(n,xs):
    if n<=0 or len(xs)==0:
        return xs
    else:
        return myDrop(n-1,xs[1:])

print myDrop(2,[1,2,3,4])
```

# 语法
## 自定义新的数据类型

使用`data`关键字定义新的数据类型

`BookInfo`是类型名，`Book`是值构造器的名字
类型名和值构造器命名不会冲突

之后的是类型的组成部分

注意：相同结构的不同类型也是区别对待的
```
data BookInfo = Book Int String [String]
    deriving(Show)

myBook = Book 1234 "my book" ["harryx","herui"]

*Main> myBook
Book 1234 "my book" ["harryx","herui"]

*Main> :t myBook
myBook :: BookInfo

*Main> :info BookInfo
data BookInfo = Book Int String [String]    -- Defined at test.hs:9:6
instance Show BookInfo -- Defined at test.hs:10:14

*Main> :t Book
Book :: Int -> String -> [String] -> BookInfo
```

## 类型别名

`type`关键字给**已存在**的类型添加别名

```
type CustomerID = Int
type ReviewBody = String

data BookReview = BookReview BookInfo CustomerID ReviewBody
    deriving(Show)

myBookReview = BookReview myBook 3 "hello harryx"

*Main> myBookReview
BookReview (Book 1234 "my book" ["harryx","herui"]) 3 "hello harryx"
```

## 代数数据类型

Bool类型是代数数据类型**（algebraic data type）**最简单的例子

`Bool`有两个值构造器，分别是`False`和`True`，用 `|` 分为两个分支

```
data Bool = False | True
```

`BookPay`类型提供三种值构造器

分别对应信用卡、支付宝、现金三种方式

不同值构造器所需的参数可能不同

```
type CardNumber = String
type AlipayNumber = String
type  Address = [String]
data BookPay = Card CardNumber
    | Alipay AlipayNumber
    | Cash Address
    deriving(Show)

*Main> :info BookPay
data BookPay = Card CardNumber | Alipay AlipayNumber | Cash Address
    -- Defined at test.hs:21:6
instance Show BookPay -- Defined at test.hs:24:14

*Main> :t Cash
Cash :: Address -> BookPay

*Main> let myPay = Cash ["China","JS"]
*Main> myPay
Cash ["China","JS"]
```

### 元组和代数数据类型

和代数类型不同，如果元组的结构相同，那么元组的类型也相同

使用上面的例子，即使结构都是一个String，类型名都是Book，仍然区分为不同类型

```
type CardNumber = String
type AlipayNumber = String
type  Address = [String]
data BookPay = Card CardNumber
    | Alipay AlipayNumber
    | Cash Address
    deriving(Eq,Show)

myPay = Alipay "123"
yourPay = Alipay "123"
hisPay = Card "123"

*Main> myPay == yourPay
True
*Main> myPay == hisPay
False

*Main> :t myPay
myPay :: BookPay

*Main> :t hisPay
hisPay :: BookPay
```

## 模式匹配

模式匹配时，会从上到下依次检查

myNot分别定义了函数对于不同输入参数的行为

```
myNot :: Bool -> Bool
myNot True = False
myNot False = True

*Main> myNot (1/=1)
True
*Main> myNot (1==1)
False
```

递归求和

这里约束类型为`Num`，只要是`Num`类型都可以
如果约束为`Integral`，只能输入整数

其中，数组会被分解为`x``xs`两部分

```
mySum :: Num a => [a] -> a
mySum (x:xs) = x + mySum xs
mySum [] = 0
```

匹配自定义的代数类型
必须指定相应类型构造器才能匹配成功

```
bookId  (Book id title  authors) = id
bookTitle (Book id title  authors) = title
bookAuthors (Book id title  authors) = authors

*Main> bookId myBook
1234
*Main> bookTitle myBook
"my book"
*Main> bookAuthors myBook
["harryx","herui"]
```

`as`模式可以保留对整体的引用

当匹配到最后一个模式时，会把整体赋值为all

```
tell :: Show a => [a] -> [Char]
tell [] = "none"
tell (x:[]) = show x
tell (x:y:[]) = show x ++ " and " ++ show y
tell all@(x:y:_) = "long " ++ show  all

*Main> tell [1,2,3,4,4]
"long [1,2,3,4,4]"
```

### 通配符

当不需要使用某个接收的参数时，可以使用通配符，编译器也不会因为参数没有使用而报错

```
bookId  (Book id _  _) = id
bookTitle (Book _ title  _) = title
bookAuthors (Book _ _  authors) = authors
```

在模式匹配时使用通配符，可以定义一个默认行为
如果之前都没匹配成功，会匹配后面的通配符

```
mySum (x:xs) = x + mySum xs
mySum _ = 0

*Main> mySum [1,2,3]
6
*Main> mySum []
0
```

### 记录语法

在创建一个数据类型时，可以指定字段名

```
data BookInfo = Book{
    bookId :: Int,
    bookTitle :: String,
    bookAuthors :: [String]
}deriving(Eq,Show)
```

在创建值时可以指定字段名

```
myBook = Book 1234 "my book" ["harryx","herui"]

yourBook = Book{
    bookId = 2345,
    bookTitle = "your book",
    bookAuthors = ["someone","the others"]
}
```

使用自带的访问器函数

```
*Main> bookId myBook
1234
*Main> bookTitle yourBook
"your book"
```

### 递归类型

其中it为上一个求值

```
data List a = Cons a (List a)
    | Nil
    deriving(Show)

*Main> Cons 0 Nil
Cons 0 Nil

*Main> Cons 1 it
Cons 1 (Cons 0 Nil)

*Main> Cons 2 it
Cons 2 (Cons 1 (Cons 0 Nil))
```

List和list的转换

```
fromList (x:xs) = Cons x (fromList xs)
fromList []     = Nil

fromCons (Cons a x) = a : (fromCons x)
fromCons Nil = []

*Main> fromList "asdasd"
Cons 'a' (Cons 's' (Cons 'd' (Cons 'a' (Cons 's' (Cons 'd' Nil)))))

*Main> fromCons (fromList "asdasd")
"asdasd"
```

使用模式匹配和递归
返回数组最大值

```
maxXs :: (Num a,Ord a) => [a] -> a
maxXs [] = error "empty"
maxXs [x] = x
maxXs (x:xs)
    | x > (maxXs xs) = x
    | otherwise = (maxXs xs)
```

Python实现

```
def maxLis(lis):
    if  len(lis) == 0:
        return 0
    if len(lis) == 1:
        return lis[0]
    if lis[0] > maxLis(lis[1:]):
        return lis[0]
    else:
        return maxLis(lis[1:])

print maxLis([1,2,3,4,5,3,2,5,7,4,2,3])
```

排序

```
mySort :: (Ord a) => [a] -> [a]
mySort [] = []
mySort (x:xs) = 
    let {
    smallone = mySort [a | a <- xs, a <= x];
    bigone = mySort [a | a <- xs, a > x];
}
    in smallone ++ [x] ++ bigone
```

使用Python实现

```
def mySort(lis):
    if lis == []:
        return []
    else:
        return mySort([i for i in lis[1:] if i <= lis[0]]) \
            + [lis[0]] \
            + mySort([i for i in lis[1:] if i > lis[0]])

print mySort([1,3,2,5,4,5,23,24,1,14,2,4124,12,21,1])
```

### 报告错误

定义一个返回数组倒数第二个值，如果数组长度太短返回错误

```
lastButOne xs = if null xs || (length xs) == 1
    then error "list too short"
    else head (drop ((length xs) -2) xs)

*Main> lastButOne [1,2,3,4,5,6,7]
6

*Main> lastButOne [1]
*** Exception: list too short
*Main> lastButOne []
*** Exception: list too short
```

### 局部变量

在函数内部，`let`把变量限制在`in`后面的表达式

```
foo = let x=1
    in ((let x = "asd" in x),x)

*Main> foo
("asd",1)
```

where和let类似，不同的是，语句定义在主句后面

```
foo = a
    where a=1
```

where语句中可以进行模式匹配

```
initials :: String -> String -> String   
initials firstname lastname = [f] ++ ". " ++ [l] ++ "."   
    where (f:_) = firstname   
          (l:_) = lastname  
```

### Case

`Case`语句

case后面是一个表达式，表达式的值为匹配目标
of后面是匹配区的开始

```
testCase value =
    case value of
        True -> False
        False -> True
```

### 门卫
类似功能的语句

```
bmiTell :: (RealFloat a) => a -> a -> String   
bmiTell weight height   
    | weight / height ^ 2 <= 18.5 
        = "You're underweight, you emo, you!"   
    | weight / height ^ 2 <= 25.0 
        = "You're supposedly normal. Pffft, I bet you're ugly!"   
    | weight / height ^ 2 <= 30.0 
        = "You're fat! Lose some weight, fatty!"   
    | otherwise                 
        = "You're a whale, congratulations!"  
```

使用局部变量改进函数

```
bmiTell :: (RealFloat a) => a -> a -> String   
bmiTell weight height   
    | bmi <= skinny
        = "You're underweight, you emo, you!"   
    | bmi <= normal
        = "You're supposedly normal. Pffft, I bet you're ugly!"   
    | bmi <= fat
        = "You're fat! Lose some weight, fatty!"   
    | otherwise                 
        = "You're a whale, congratulations!"  
    where {
        bmi = weight / height ^ 2;
        (skinny, normal, fat) = (18.5, 25.0, 30.0)
}
```

### 中缀函数

```
a `myplus` b = a+b

*MyModule> 1 `myplus` 2
3
*MyModule> myplus 1 2
3
```

# 柯里函数

使用参数不完全的函数定义新函数

```
Prelude> let conpTen = max 10
Prelude> conpTen 9
10
```

# 高阶函数

将函数作为参数调用或返回

这里将函数f递归调用，作用在两个数组

真正调用的函数类型可以和函数参数不同
不一定是 a -> b -> c

```
myCons :: a -> a -> [a]
myCons x y = x:[y]

myZip :: (a -> b -> c) -> [a] -> [b] -> [c]
myZip _ _ [] = []
myZip _ [] _ = []
myZip f (x1:x1s) (x2:x2s) = (f x1 x2) : (myZip f x1s x2s)

*Main> myZip myCons [1,2,3,4] [3,4,5,6]
[[1,3],[2,4],[3,5],[4,6]]
```

Python实现

```
def myZip(f,lis1,lis2):
    if lis1 == [] or lis2 == []:
        return []
    else:
        return [f(lis1[0],lis2[0])] + myZip(f,lis1[1:],lis2[1:])

lis1=[1,1,2,3]
lis2=[2,2,4,5]

def myCons(a,b):
    return [a,b]

print myZip(myCons,lis1,lis2)
```

## 匿名函数

Haskell的lambda用\表示

```
myFlip :: (a -> b -> c) -> b -> a -> c
myFlip f x y = f y x 

*Main> myFlip (\a b -> a:[b]) 1 2
[2,1]
```

## 折叠

`foldl`实现左折叠
`foldr`右折叠
将一个二元函数作为参数

```
myFoldlSum :: Num a => [a] -> a
myFoldlSum xs = foldl (\acc x -> acc + x) 0 xs
```

柯里化可以省略参数
`\acc x -> acc + x`和`+`是等价的

```
myFoldlSum :: Num a => [a] -> a
myFoldlSum = foldl (+) 0
```

使用`:`将数组反向拼接

```
myFoldlRev :: [a] -> [a]
myFoldlRev = foldl (\acc x -> x:acc) []

*Main> myFoldlRev [1,2,3]
[3,2,1]
```

`foldl1``foldr1`把折叠的第一个值作为初始值(数值)

```
myFoldlSum :: Num a => [a] -> a
myFoldlSum = foldl1 (+)
```

## `$`符号

```
($) :: (a -> b) -> a -> b   
f $ x = f x
```

`$`相当于在后面添加了括号

```
*Main> sum $ map (+1) [1,2,3,4]
14
*Main> sum ( map (+1) [1,2,3,4])
14
```

## 函数组合

```
(.) :: (b -> c) -> (a -> b) -> a -> c   
f . g = \x -> f (g x)
```

# 模块
## 装载模块

```
Prelude> import Data.List
Prelude Data.List> 
```

在ghci命令行也可以使用`:m`装载模块

```
Prelude> :m Data.List
Prelude Data.List> 
```

只装载某个函数
```
import Data.List  (nub)
```

去除某个函数
```
import Data.List  hiding (nub)
```

避免命名冲突
```
import qualified Data.List

*Main> Data.List.nub [1,2,12,1]
[1,2,12]
```

自定义模块名
```
import qualified Data.List as L

*Main> L.nub [1,2,3,3,2]
[1,2,3]
```

示例
```
import Data.List  

numUniques :: (Eq a) => [a] -> Int   
numUniques = length . nub

*Main Data.List> numUniques [1,2,3,2,1,3]
3
```
