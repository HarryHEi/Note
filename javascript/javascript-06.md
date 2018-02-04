---
title: javascript-06 动态创建标记
date: 2016-04-28 19:57:24
tags: [javascript]
---

# 动态创建标记
## 传统方法
### document.write
document对象的write()方法可以方便便捷地把字符串插入到文档里
```
document.write("<p>hello harryx</p>")
```
但是这种方法不够优雅，很难将脚本与文档分离，应该避免使用

### innerHTML属性
innerHTML可以用来读写某个标签元素里面的内容，下例可以读出id为description标签的内容为picture-0
```
testdiv = document.getElementById("description")
alert(testdiv.innerHTML)
```
innerHTML属性不能处理细节，适用于把大段的文本插入HTML
```
testdiv = document.getElementById("description")
testdiv.innerHTML = "<p><em>picture-</em> 1</p>"
```
一旦使用了innerHTML，元素内容会被全部替换，相比较write()方法，这个属性更方便实用

## DOM方法
DOM所包含的信息与文档里面的信息一一对应，通过DOM可完成读取和更新文档内容

### createElement方法
在文档里面放一个空白的标签，设置id为testdiv
```
<div id="testdiv">
</div>
```
下面要在这个空白标签插入一个p元素节点，需要两步来完成

1. 创建一个新的元素
2. 把这个新元素插入节点树

第一个步骤需要使用createElement来完成，创建一个新元素标签然后赋值给变量para
```
var para = document.createElement("p")
```

### appendChild方法
现在已经创建了一个新标签元素，然后需要把这个元素插入到文档的节点，使用appendChild方法
先找出要插入这个新元素的节点位置，就是之前id为testdiv的空标签
```
var testdiv = document.getElementById("testdiv");
```
然后插入
```
testdiv.appendChild(para);
```

### createTextNode方法
现在已经创建了一个新元素并插入到文档的节点中，但是p是个空白元素，下面是在p元素中放入文本
使用createTextNode创建文本节点，如果查看txt的nodeType，返回值是3(文本节点)
```
var txt = document.createTextNode("hello harryx")
```
然后把这个文本节点插入到p元素(标签元素)
```
para.appendChild(txt)
```
然后就实现了使用DOM插入一段文本，所有代码如下，相比较使用innerHTML属性而言，更细节化
```
var para = document.createElement("p");
var testdiv = document.getElementById("testdiv");
testdiv.appendChild(para);
var txt = document.createTextNode("hello harryx")
para.appendChild(txt)
```
类似的DOM方法还有
在已有的元素之前插入一个新元素
```
insertBefore(新元素, 目标元素)
```
自定义一个insertAfter函数
```
function insertAfter(newElement, targetElement)
{
    var parent = targetElement.parentNode; //父元素
    if (parent.lastChild == targetElement) //如果目标元素是最后一个元素
    {
        parent.appendChild(newElement); //插入到父元素最后面
    }
    else
    {
        parent.insertBefore(newElement, targetElement.nextSibling); //否则插入到目标元素后面一个兄弟元素后面
    }
}
```
其中使用了一个新属性：nextSibling属性，某个元素的下一个兄弟元素

## 完全分离
通过使用JavaScript可以创建标签，而不需要在文档中编写，下面将结构、样式和行为完全分离

在HTML文档中的标签内容如下
```
<div id="preview">
    <table >
        <tr>
            <td class="change"><a href="#" id="pre" >上一个</a>
            <td class="change"><a href="#" id="next">下一个</a>
        </tr>
    </table>
</div>
<div class="message"><a href="#" class="popup">查看原图</a></div>
<p id="description" class="message"><em>picture-</em> 0</p>
<img id="picture" src="./static/pic2/pic0.jpg" width="600px" alt="到头了">
```
在插入之前，文档是这样的
```
<div id="content">
    //.....
    <div id="buding">
        <a href="https://www.bilibili.com/"><img src="./static/image/buding.gif" width="120px" alt="buding" /></a>
    </div>
</div>
```
先找到id为content的div元素，然后在id为buding的子元素之前插入一个id为main的div元素，然后依次在id为main的元素内插入其他元素
代码如下
```
function addPreview ()
{
    var allContent = document.getElementById("allcontent");
    var buDing = document.getElementById("buding");
    var parent = document.createElement("div");
    parent.setAttribute("id", "main");
    allContent.insertBefore(parent, buDing);

    var previewDiv = document.createElement("div");
    previewDiv.setAttribute("id", "preview");
    parent.appendChild(previewDiv);
    var table = document.createElement("table");
    previewDiv.appendChild(table);
    var tr = document.createElement("tr");
    table.appendChild(tr);
    td1 = document.createElement("td");
    td1.setAttribute("class", "change");
    td2 = document.createElement("td");
    td2.setAttribute("class", "change");
    tr.appendChild(td1);
    tr.appendChild(td2);
    preA = document.createElement("a");
    preA.innerHTML = "上一个";
    preA.setAttribute("href", "#");
    preA.setAttribute("id", "pre");
    td1.appendChild(preA);
    nextA = document.createElement("a");
    nextA.innerHTML = "下一个";
    nextA.setAttribute("href", "#");
    nextA.setAttribute("id", "next");
    td2.appendChild(nextA);

    var popDiv = document.createElement("div");
    popDiv.setAttribute("class", "message");
    parent.appendChild(popDiv);
    var popA = document.createElement("a");
    popA.setAttribute("href", "#");
    popA.setAttribute("class", "popup");
    popA.innerHTML = "查看原图";
    popDiv.appendChild(popA);

    desP = document.createElement("p");
    desP.setAttribute("id", "description");
    desP.setAttribute("class", "message")
    desP.innerHTML = "<em>picture-</em> 0";
    parent.appendChild(desP);

    imgPre = document.createElement("img");
    imgPre.setAttribute("id", "picture");
    imgPre.setAttribute("src", "./static/pic2/pic0.jpg");
    imgPre.setAttribute("alt", "到头了");
    parent.appendChild(imgPre);
}
```
最后使用addLoadEvent导入
```
addLoadEvent(addPreview);
```

## Ajax
Ajax用于概括异步加载页面内容的技术，依赖于JavaScript

### XMLHttpRequest 对象
Ajax技术的核心是XMLHttpRequest对象，这个对象充当浏览器中脚本(客户端)与服务器之间的中间人的角色
JavaScript通过这个对象可以发送请求，也能处理请求

不同的浏览器XMLHTTP对象不尽相同，不同版本的IE浏览器XMLHTTP对象也不完全相同

查看XMLHttpRequest对象有没有定义，如果定义了返回defined
```
if (typeof XMLHttpRequest == "undefined")
{
    alert("undefined")
}
else
{
    alert("defined")
}
```
或者一个个尝试
```
function getHTTPObject()
{
    if (typeof XMLHttpRequest == "undefined")
    {
        XMLHttpRequest = function()
        {
            try
            {
                return new ActiveXObject("Msxml2.XMLHTTP.6.0");
            }
            catch (e) {}
            try
            {
                return new ActiveXObject("Msxml2.XMLHTTP.3.0");
            }
            catch (e) {}
            try
            {
                return new ActiveXObject("Msxml2.XMLHTTP");
            }
            catch (e) {}
            return false;
        }
    }
    return new XMLHttpRequest();
}
```
调用
```
var request = getHTTPObject();
```
XMLHttpRequest对象有许多方法，其中最有用的是open方法，用来指定服务器上将要访问的文件，指定请求类型：GET、POST、SEND，这个方法的第三个参数用于指定请求是否以异步方式发送和处理

readyState属性有五种可能值

+ 0 表示没有初始化
+ 1 表示正在加载
+ 2 表示加载完毕
+ 3 表示正在交互
+ 4 表示完成

```
function getNewContent() 
{
    var request = getHTTPObject(); //创建XMLHTTP对象
    if (request) //如果成功创建对象
    {
        request.open("GET", "example.txt", true); //发送一个“GET”请求，要求服务器异步发送一个名为example.txt的文件
        request.onreadystatechange = function () //onreadystatechange是一个事件处理函数，服务器回送响应时执行
        {
            if (request.readyState == 4) //服务器会在不同阶段更新readyState的值
            {
                //在id为“new“的div元素创建一个子元素p显示文档
                var para = document.createElement("p");
                var txt = document.createTextNode(request.responseText); //从responseText属性理获得文本数据
                para.appendChild(txt);
                document.getElementById("new").appendChild(para);
            }
        };
        request.send(null); //发送一个请求
    }
    else //浏览器不支持Ajax
    {
        alert("不支持")
    }
}
addLoadEvent(getNewContent);
```
如果在事件处理函数内加一个输出0，在整个函数最后加一个输出1，执行后会发现，先输出1，然后再输出0
说明函数不会等待服务器的请求，而是会继续执行，这就是异步加载
```
function getNewContent() 
{
    var request = getHTTPObject();
    if (request)
    {
        request.open("GET", "./static/example.txt", true);
        request.onreadystatechange = function ()
        {
            if (request.readyState == 4)
            {
                alert("0");
                var para = document.createElement("p");
                var txt = document.createTextNode(request.responseText);
                para.appendChild(txt);
                document.getElementById("new").appendChild(para);
            }
        };
        request.send(null);
    }
    else
    {
        alert("不支持")
    }
    alert("1")
}
```
