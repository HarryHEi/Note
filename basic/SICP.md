---
title: SICP 笔记(一)
date: 2016-05-18 16:59:34
tags: [sicp]
---

# 简述
《计算机程序的构造和详解》(SICP)的读书笔记，习题内容用python实现

# 计算
## 1.5
区别正则序和应用序
```
(define (p) (p))
(define (test x y))
    (if (= x 0))
        0
        y))

求值
(test 0 (p))
```

应用序中，解释器会首先对运算符和各对象求值
正则序中，解释器先不求出运算对象的值，只有实际需要时才去求值

上述程序中(p)会不断调用自身，从而陷入死循环
应用序中，程序会执行(p)并陷入死循环
正则序中，程序会因为if语句不执行(p)，而正常输出0

## 1.6
使用cond自定义一个if语句
```
(define (new-if predicate then-clause else-clause)
    (cond (predicate then-clause)
        (else else-clause)))
```
和一般的if语句不同的是，new-if只是普通函数
在应用序中，new-if语句的每个参数都会被求值

上述例子中，then-clause 和 else-clause都会被执行，和预期不同

## 树形递归
用递归方法计算斐波那契数列
```
#!/usr/bin/env python
#-*coding:utf-8-

def fib(n):
    if (n==0):
        return 0
    if (n==1):
        return 1
    if (n>1):
        return fib(n-1)+fib(n-2)

print fib(35)
```
进行了多次冗余计算
```
9227465
[Finished in 7.1s]
```

使用迭代方法计算
```
#!/usr/bin/env python
#-*coding:utf-8-

def fib(n):
    def fib_inter(a,b,count): # a+b->a a->b
        if (count==0):
            return b
        else:
            return fib_inter(a+b,a,count-1)
    return fib_inter(1,0,n) # b初始化为0，a初始化为1

print fib(5)
```
非常迅速
```
9227465
[Finished in 0.0s]
```

换零钱实例
总数为a的现金兑换成n种硬币的方式总数
    = a现金不用某一种硬币的兑换的方式数
    + (a-d)现金用了这种硬币的兑换的方式数(d为这个硬币值)
```
#!/usr/bin/env python
#-*coding:utf-8-

def count_change(amount):
    def first_deno(kinds_of_coins):
        if (kinds_of_coins==1):
            return 1
        elif (kinds_of_coins==2):
            return 5
        elif (kinds_of_coins==3):
            return 10
        elif (kinds_of_coins==4):
            return 25
        elif (kinds_of_coins==5):
            return 50
    
    def cc(amount,kinds_of_coins):
        if (amount==0): 
            return 1
        elif (amount<0 or kinds_of_coins==0):
            return 0
        else:
            return cc(amount,kinds_of_coins-1)+cc(amount-first_deno(kinds_of_coins),kinds_of_coins)

    return cc(amount,5)

print count_change(100)
```

## 1.12
`n<3   f(n)=n`
`n>=3 f(n)=f(n-1)+2f(n-2)+3f(n-3)`

使用递归
```
def f(n):
    if (n<3):
        return n
    else:
        return f(n-1)+2*f(n-2)+3*f(n-3)
print f(10)
```

使用迭代
```
def f(n):
    def f_inter(a,b,c,count):
        if count<3:
            return c
        else:
            return f_inter(b,c,3*a+2*b+c,count-1)
    return f_inter(0,1,2,n)
print f(10)
```

## 1.12
帕斯卡三角形

使用递归
根据除了第一和第一行外每个元素`f(x,y) = f(x-1,y-1) + f(x-1,y)`

```
def f(x,y):
    if y==0 or x==y or x==1:
        return 1
    else:
        return f(x-1,y-1)+f(x-1,y)
print f(10,5)
```

使用迭代
根据公式 `f(x,y) = x! / (y! * (x-y)!)`

```
def f(x,y):
    def f_inter(x):
        if x==0:
            return 1
        f_inter_result = 1
        for i in range(x):
            f_inter_result *= i+1
        return f_inter_result
    return f_inter(x)/(f_inter(y)*f_inter(x-y))
print f(10,5)
```

## 1.15
x足够小时可以使用`sinx=3 sin(x/3) - 4 sin3(x/3)`

```
def sine(x):
    def f(x): # 角度足够小时使用这个函数求值
        return 3*x-4*(x**3)
    def sines(angle): # 递归求值
        if abs(angle)<0.1:
            return angle
        else:
            return f(sines(angle/3))
    return sines(x)
print sine(12.15)
```

sine(12.15) 5次调用 f(x)
每增加一倍，运行次数会增加一次

## 求幂
一般的求幂，相比较直接使用递归计算 `x*(x*(x*(x*(x(...)))))`
更快的做法是连续求方
n为偶数时 `x^n = (x^(n/2))^2`
n为奇数时 `x^n = (x^(n-1))*x`

```
def exp(x,n):
    if n==0:
        return 1
    elif n%2==0:
        return exp(x,n/2)**2
    elif n%2!=0:
        return exp(x,n-1)*x

print exp(2,10)
```

## 1.16
使用迭代法求幂
n为偶数时 `x**2 -> x ; n/2 -> n ; a -> a`
n为奇数时 `x -> x ; n-1 -> n ; a*x -> a`
`a*(b^n)` 的值是不变的

```
def exp(x,n):
    def ex(x,n,a):
        if n==0:
            return a
        elif n%2==0:
            return ex(x**2,n/2,a)
        elif n%2!=0:
            return ex(x,n-1,a*x)
    return ex(x,n,1)

print exp(2,10)
```

## 1.17
只使用double(将一个数X2)，和halve(将一个数除以2)，定义一个快速的乘法
例如：
`5*8 = 5*4*2`
`5*7 = 5*6+5`

```
def doubles(a):
    return a*2

def halv(a):
    return a/2

def mut(a,b):
    if b==0:
        return 0
    elif b%2==0:
        return doubles(mut(a,halv(b)))
    elif b%2!=0:
        return mut(a,b-1)+a

print mut(2,8)
```

## 1.18
使用迭代
```
def doubles(a):
    return a*2

def halv(a):
    return a/2

def mut(a,b):
    def mut(a,b,project):
        if b==0:
            return project
        elif b%2==0:
            return doubles(mut(a,halv(b),project))
        elif b%2!=0:
            return mut(a,b-1,a+project)
    return mut(a,b,0)

print mut(5,8)
```

# 过程
## 过程参数
定义一个过程变量，从外部传递参数
```
def sums(term,a,b):
    if a>b:
        return 0
    else:
        return term(a)+sums(term,a+1,b)

def powers(a):
    return a*a

print sums(powers,1,3)
```

## 使用lambda构造过程
```
def sums(term,a=1,b=2):
    if a>b:
        return 0
    else:
        return term(a)+sums(term,a+1,b)

print sums(lambda x:x*2,a=1,b=3)
```
也可以返回过程
```
def fun():
    return lambda x:x+1

print fun()(2)
```

# 数据抽象
## 序对
使用cons将两个对象粘接在一起
然后使用car和cdr获得其中一个对象
```
1 ]=> (display (car (cons 1 2)))
1

1 ]=> (display (cdr (cons 1 2)))
2
```
完全使用过程实现序对
cons(x,y)的返回值是一个过程(函数)
```
def cons(x,y):
    return lambda z : x if z else y

def car(c):
    return c(1)

def cdr(c):
    return c(0)

print car(cons(1,2))
```
为了效率，在LISP中实际上是直接使用数据实现序对

### 序列
序对的嵌套可以形成序列，其中使用 `'()` 定义空
```
(define test-cons-list (cons 1 (cons 2 (cons 3 (cons 4 '())))))

1 ]=> test-cons-list
;Value 21: (1 2 3 4)
```
使用list方法也可以构造
```
(define test-lis (list 1 3 5 7 2 4 6 8))
```
使用car cdr遍历表
cadr 是 (car (cdr ...)) 的简写
定义一个遍历序列操作
```
(define (list-ref items n)
    (if (= n 0)
        (car items)
        (list-ref (cdr items) (- n 1))))
```
(null? ...) 基本操作，检查参数是不是空
定义一个检查序列长度的操作
递归操作
```
(define (length items)
    (if (null? items)
        0
        (+ 1 (length (cdr items)))))
```
迭代操作
```
(define (length items)
    (define (length-inter items-inter count)
        (if (null? items-inter)
            countdd
            (length-inter (cdr items-inter) (+ 1 count))))
    (length-inter items 0))
```

### 2.17
获取最后一个元素的值
使用递归方法实现
```
(define (last-pair items)
    (if (null? (cdr items))
        items
        (last-pair (cdr items))))
```

### 2.18
倒序
使用迭代
```
(define (reverse items)
    (define (reverse-inter items-inter result)
        (if (null? items-inter)
            result
            (reverse-inter (cdr items-inter)
                (cons (car items-inter) result))))
    (reverse-inter items '()))
```

### 对表的映射
对表的映射实现的是从元素表到结果表的变换
```
1 ]=> test-lis
;Value 14: (1 3 5 7 2 4 6 8)

1 ]=> (map + test-lis test-lis)
;Value 13: (2 6 10 14 4 8 12 16)
```

## 层次性结构
计算叶子数
```
(define (count-leaves items)
    (cond ((null? items) 0)
        ((not (pair? items)) 1)
        (else (+ (count-leaves (car items))
            (count-leaves (cdr items))))))
```

## 快速排序
用python实现快速排序
```
def mysort(a):
    def sort(a,p,q):
        x=a[p]
        i=p
        j=p+1
        while j<len(a):
            if a[j]<=x:
                i=i+1
                temp=a[j]
                a[j]=a[i]
                a[i]=temp
            j=j+1
        temp=a[p]
        a[p]=a[i]
        a[i]=temp
        return i

    def intersort(a,p,q):
        if p<q:
            r = sort(a,p,q)
            intersort(a,p,r-1)
            intersort(a,r+1,q)
    intersort(a,0,len(a)-1)
    return a
```
