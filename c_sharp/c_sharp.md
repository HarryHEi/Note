---
title: C#面向对象
date: 2017-03-29 10:35:14
tags: [c_sharp]
---

抽空研究了一下C#，稍微整理了下。

# 类

和C++相比，C#省去了指针（内存）的直接操作，类也有一些细节上的变化。

和C++类似，C#使用`virtual`和`override`关键字继承类方法，使用`new`关键字隐藏方法

```
class Animal
{
    public int num_of_legs;
    public string food;
    public Animal(int num, string f)
    {
        num_of_legs = num;
        food = f;
    }
    public virtual string say()  // 使用virtual关键字创建虚函数
    {
        return "...";
    }
}

class Dog : Animal
{
    public Dog(int num, string f):
        base(num, f){}
    public override string say()  // 这里的override关键词不能省略
    {
        return "汪!!";
    }
}
```

子类使用base关键字访问基类

```
class Cat : Animal
{
    public Cat(int num, string f):
        base(num, f){}
    public override string say()
    {
        return "喵?";
    }
}

class BigCat : Cat
{
    public BigCat(int num, string f):
        base(num, f){}
    public override string say()
    {
        return base.say() + "喵?喵?";
    }
}
```

# 接口

C#有个叫`interface`（接口）的东西（作用相当于契约），看上去和类差不多，但是接口不能被实例化

接口不能定义变量，只能定义某种方法

```
interface IAnimal
{
    int num_of_legs { get; }  // 定义属性方法
    string food { get; }
    string say();
}

class Animal : IAnimal
{
    public int num_of_legs { get; }
    public string food { get; }
    public Animal(int num, string f)
    {
        num_of_legs = num;
        food = f;
    }
    public virtual string say()
    {
        return "...";
    }
}
```

可以通过引用操作接口

```
Bird bird = new Bird(2, "rice", 10);  // 一个Bird实例
IAnimal animal = bird;  // IAnimal引用
MessageBox.Show(animal.say());
```

可以通过`is`查看是否实现了接口

```
BigCat big_cat = new BigCat(4, "fish");
if(big_cat is IAnimal)
	MessageBox.Show(big_cat.say());
```

C#允许类使用`as`向下强制转换

如果向下转换失败，`as`语句返回null

```
Animal animal = new BigCat(4, "fish");  // 把BigCat向上强制转换成Animal
if (animal is BigCat)  // 通过is判断其是否实现了BigCat
{
    BigCat big_cat = animal as BigCat;  // 向下强制转换
    MessageBox.Show(big_cat.say());
}
```

接口的继承

接口也可继承方法，然后建立新类实现方法

```
interface IAnimal
{
    int num_of_legs { get; }
    string food { get; }
    string say();
}

interface IFlyAnimal : IAnimal
{
    int size_of_wings { get; }
}

class Bird : IFlyAnimal
{
    public Bird(int num, string f, int s)
    {
        num_of_legs = num;
        food = f;
        size_of_wings = s;
    }
    public string food { get; }
    public int num_of_legs { get; }
    public int size_of_wings { get; }

    public virtual string say()
    {
        return "...";
    }
}
```

实现新类的时候可以同时继承基类和接口

```
class Bird : Animal, IFlyAnimal
{
    public Bird(int num, string f, int s) :
        base(num, f)
    {
        size_of_wings = s;
    }
    public int size_of_wings { get; }
    public override string say()
    {
        return "啾！";
    }
}
```

接口作用

接口用起来就像一个指针，它可以用来表示实现了某个接口的所有类。

# 抽象类

抽象类和类相似，可以继承、储存变量或者实现方法，但是和接口一样，不能实例化

```
abstract class Animal : IAnimal
{
    public int num_of_legs { get; set; }
    public string food { get; set; }
    public Animal(int num, string f)
    {
        num_of_legs = num;
        food = f;
    }
    public virtual string say()
    {
        return "...";
    }
}
```

# 实现比较方法

C#的类先继承接口然后在实现方法，下面实现一个比较方法

Card类先继承`IComparable<Card>`接口，然后实现`CompareTo`方法

```
class Card : IComparable<Card>
{
    private Suit suit;
    private Value value;
    public Card(Suit suit, Value value)
    {
        this.suit = suit;
        this.value = value;
    }

    // 先比较花色然后比较数字大小
    public int CompareTo(Card other)
    {
        if (suit == other.suit)
        {
            if (value > other.value)
                return 1;
            else if (value < other.value)
                return -1;
            else
                return 0;
        }
        else if (suit > other.suit)
            return 1;
        else
            return -1;
    }

    public string get_name()
    {
        return suit + " " + value;
    }
}
```
