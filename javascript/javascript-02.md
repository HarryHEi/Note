---
title: javascript-02 DOM
date: 2016-04-26 10:58:40
tags: [javascript]
---

# DOM
## 文档：D(Document)
当创建一个网页并加载到Web浏览器中时，DOM就在幕后悄然而生，把网页文档转化为一个文档对象。

## 对象：O(Object)
JavaScript对象分为三种类型

+ 用户自定义对象：由程序员自行创建的对象
+ 内建对象：内建在JavaScript的对象，如Array，Math等
+ 宿主对象：由浏览器提供的对象

对编写脚本来说非常重要的宿主对象中最基础的对象是window对象

window对象对应着浏览器窗口本身，这个对象的属性和方法统称为BOM(浏览器对象模型)，或者称为窗口对象模型

BOM提供window.open和window.blur等方法，这些方法负责窗口的弹出和下拉等操作

document对象的主要功能是处理网页内容

## 模型：M(Model)
DOM代表模型(Model)，或者也可以说代表地图(Map)，含义是某种事物的表现形式

DOM把一份文档表示为一棵树，一棵家谱树(节点树)，可以把各元素的关系简明表示出来

## 节点
节点(node)这个词是个网络术语，它表示网络中的一个连接点，一个网络就是节点构成的集合

DOM也是由节点构成的集合，只不过节点是文档树上的树枝、树叶而已

### 元素节点
DOM的原子是**元素节点**(element node)

标签的名字就是元素的名字：例如文本段落元素"P"，无序清单元素"ul"，列表项元素"li"

其中没有被其他元素包含的元素"html"被称为根元素

### 文本节点
元素节点内部包含的节点为文本节点，不过并不是每个元素节点都包含文本节点

### 属性节点
属性节点用来对元素做出更具体的描述，例如几乎所有元素都有一个title属性
```
<p title="message">welcome!</p>
```
并非所有的元素都包含某个属性，但是所有属性都被元素包含

### CSS
CSS(层叠样式表)告诉浏览器应该如何显示一份文档的内容

### 获取元素
有三种DOM方法可以获得元素节点，分别是通过元素ID、通过标签名字和通过类名字
#### getElementById
顾名思义就是通过元素ID获取元素节点
```
document.getElementById("myname");
```
对应着id为“myname”的标签div
```
#...
    <div id="myname">
        Javascript
    </div>
#...
```
如果使用typeof观察返回值类型，可以直到类型为对象(Object)
```
alert(typeof document.getElementById("myname"));
```

#### getElementsByTagName
更具标签的名字获取元素节点和getElementById()类似，不过返回的是一个数组
```
getElementsByTagName("li");
```
可以通过length属性查看元素个数
```
alert(document.getElementsByTagName("p").length);
```
可以把读取的元素节点赋值给一个变量
```
ps = document.getElementsByTagName("p");
```
可以把通配符作为参数，匹配所有元素节点
```
ps = document.getElementsByTagName("*");
```
可以把getElementById()和getElementsByTagName结合起来用
先找出id属性为allcontent的元素节点，然后找出该节点内的所有元素并统计个数
```
var cont = document.getElementById("allcontent");
var items = cont.getElementsByTagName("*");
alert(items.length);
```

#### getElementsByClassName
HTML5 DOM新增的方法，通过class属性访问元素
```
var res = document.getElementsByClassName("linkp");
alert(res.length);
```
同样的，也可以结合使用
```
var cont = document.getElementById("allcontent");
var res = cont.getElementsByClassName("linkp");
alert(res.length);
```
如果浏览器不支持这个方法，可以自己实现一个类似功能的函数
```
function getElementsByClassName (node, classname) 
{
    if (node.getElementsByClassName)
    {
        return node.getElementsByClassName(classname);
    }
    else
    {
        var results = new Array();
        var elems = node.getElementsByTagName("*")
        for (var i=0; i<items.length; i++)
        {
            if (items.className.indexof(classname) != -1)
            {
                results[results.length] = elems[i];
            }
        }
        return results;
    }
}
```
第一个参数node表示开始节点，第二个参数是类名，如果节点已经存在了getElementsByClassName函数，直接返回列表值，如果不存在，新函数会循环遍历所有标签，查找带有相应类名的元素
```
var cont = document.getElementById("allcontent")
var res = getElementsByClassName(cont, "linkp")
alert(res.length)
```
## 获取和设置属性
得到元素之后，就是获取属性节点的值

### getAttribute
getAttribute是一个函数，只有一个参数，查询的属性的名字

getAttribute不属于document对象，只能通过元素节点对象调用

"class"属性的值，如果没有这个属性，会返回null
```
var ps = document.getElementsByTagName("p")
for (var i=0; i<ps.length; i++)
{
    alert(ps[i].getAttribute("class"))
}
```

### setAttribute
setAttribute允许对属性节点的值做出修改，和getAttribute相同，也只能用于元素节点

改变一个img元素的寬和高
```
var bud = document.getElementById("buding")
var gif = bud.getElementsByTagName("img")
gif[0].setAttribute("width","300px")
gif[0].setAttribute("height","300px")
```
如果设置的属性之前存在，会直接覆盖掉，虽然属性已经改变了，但是如果查看网页源码，属性并没有改变，所以DOM实现的是先静态显示再动态刷新，动态刷新不会影响静态的内容
