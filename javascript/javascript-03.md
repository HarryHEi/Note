---
title: javascript-03 图片库
date: 2016-04-27 10:11:01
tags: [javascript]
---

# JavaScript图片库
## 标记
添加一个img标记，显示一个图片，添加一个独有的id，然后通过id查找到这个元素之后可以通过更改src参数更改显示的图片
```
<img id="picture" src="./static/picture/pic1.jpg" width="600px">
```

## JavaScript
展示一个图片的函数可以这样写，通过id：picture寻找到元素标签，然后使用setAttribute()函数更改src属性
```
function showPic (whichpic) 
{
    var pic = document.getElementById("picture");
    pic.setAttribute("src",whichpic);
}

show_pic("./static/picture/pic32.jpg")
```
也可以不使用DOM来解决，可以直接修改大部分元素的属性
```
function showPic (whichpic) 
{
    var pic = document.getElementById("picture");
    pic.src=whichpic; //直接修改元素的src属性
}
showPic("./static/picture/pic33.jpg")
```

## 应用JavaScript
### 事件处理函数
事件处理函数的作用是在特定事件发生时调用JavaScript代码。
例如，鼠标悬停某个元素上方时出发动作使用onmouseover事件处理函数；
鼠标指针离开某个元素时触发动作使用onmouseout事件处理函数；
用户点击某个链接时触发一个动作使用onclick事件处理函数。

当用户点击某个链接时，读取这个读片链接并预览出来，把函数改为这样
```
function showPic (whichpic) 
{
    var source = whichpic.getAttribute("href")
    var pic = document.getElementById("picture");
    pic.setAttribute("src",source);
}
```
把oneclick事件处理函数嵌入到一个链接时，如果要把某个链接本身作为参数传递，使用this关键字传递对象

添加事件处理函数的语法如下：
```
event = "JavaScript"
```
在一对引号之间可以放入任意数量的JavaScript语句，只要用分号隔开
```
onclick = "showPic(this)"
```
不过当用户点击链接时还会调用本来的行为，页面会跳到点开的页面中，因此要阻止这个默认行为

测试一下事件处理函数的工作机制
```
<a href="./static/picture/pic5.jpg" onclick="return false;">test</a>
```
当点击这个链接时，onclick返回值是false，事件处理函数就认为这个链接没有被点击，所以链接不会被触发
所以在后面加上一句return false，就不会触发链接
```
<a href="./static/picture/pic1.jpg" onclick="showPic(this);return false;">test</a>
```
如果static里面的图片是按照一定顺序命名，利用一个判断语句，可以添加上一页和下一页的按钮索引图片
```
var num=0

function showPic (whichpic) 
{
    var source = whichpic.getAttribute("href");
    if (source == "pre")
    {
        if (num != 0)
        {
            num --;
            source = "./static/picture/pic" + num + ".jpg";
        }
    }
    if (source == "next")
    {
        num ++;
        source = "./static/picture/pic" + num + ".jpg";
    }
    var pic = document.getElementById("picture");
    pic.setAttribute("src",source);
}
```
在html文件中调用
```
<li>
    <a href="pre" onclick="showPic(this);return false;">上一个</a>
</li>
<li>
    <a href="next" onclick="showPic(this);return false;">下一个</a>
</li>
```

### childNodes属性
childNodes用于获得一个元素的所有子元素的集合，下面是一个获得body所有子元素个数的函数
window.onload事件处理函数代表在页面加载时执行
```
function countBodychildren ()
{
    var body_element = document.getElementsByTagName("body")[0];
    alert(body_element.childNodes.length);
}

window.onload = countBodychildren;
```
返回的结果会比想象中多，因为childNodes不仅返回元素节点，文档的每样东西都是一个节点

### nodeType属性
每个节点都有一个nodeType属性.nodeType的值是一个数字

+ 元素节点的nodeType属性值为1
+ 属性节点的nodeType属性值为2
+ 文本节点的nodeType属性值为3

如果把上个函数的语句改成输出nodeType，可以发现输出是1
```
alert(body_element.nodeType);
```

### 在标记里添加表述
当点击一个图片链接时，不仅更换成目标链接的图片预览，还要更换文本描述，文本表述来自链接元素的title属性
```
var text = whichpic.getAttribute("title")
```
在html文档中添加一个文本表述，id为description
```
<p id="description">choose a picture</p>
```
通过id找到这个元素节点
```
var des = document.getElementById("description")
```
下面是切换文本

### nodeValue属性
nodeValue属性可以得到或者设置一个节点的值，但是这个值并不知元素节点的文本
```
alert(des.nodeValue) //返回null
```
真正的文本在p元素的第一个子节点的nodeValue里面
```
alert(des.childNodes[0].nodeValue) //返回choose a picture
```
也可以修改文本
```
des.childNodes[0].nodeValue = text
```

### firstChild和lastChild属性
数组元素childNodes[0]有个更直观的同义词，即firstChild，node.firstChild和node.childNodes[0]是等价的

与此相对的，lastChild代表数组的最后一个元素
```
function showPic (whichpic)
{
    var source = whichpic.getAttribute("href");
    var text = whichpic.getAttribute("title");
    var pic = document.getElementById("picture");
    var des = document.getElementById("description");
    des.firstChild.nodeValue = text;
    pic.setAttribute("src", source);
}
```
