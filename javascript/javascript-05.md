---
title: javascript-05
date: 2016-04-28 16:31:35
tags: [javascript]
---

# 分离HTML和JavaScript
将之前的图片预览脚本的HTML和JavaScript代码分离，不在HTML写JavaScript脚本，而是像CSS一样，在外部脚本文件给HTML文档添加脚本功能

首先在上下链接的标签添加class属性
```
<li>
    <a href="pre" class="preview" >上一个</a>
</li>
<li>
    <a href="next" class="preview" >下一个</a>
</li>
```
然后在JavaScript脚本添加功能，原本JavaScript的脚本函数不需要更改
```
function preparePreview ()
{
    if (!document.getElementsByTagName) return false;
    var links = document.getElementsByTagName("a");
    for (var i=0; i<links.length; i++)
    {
        if (links[i].getAttribute("class") == "preview")
        {
            links[i].onclick = function()
            {
                changePic(this);
                return false;
            }
        }
    }
}
window.onload = preparePreview;
```

# 添加事件处理函数
现在只有两个链接标签，如果有多个标签元素需要添加功能，逐个添加class会比较麻烦
可以改进一下，可以在父标签添加一个独一无二的id，然后再遍历父标签的子标签添加功能

重写preparePreview函数，能够遍历id为preview标签元素所有链接标签，并且添加功能
```
function preparePreview ()
{
    //对象测试
    if (!document.getElementsByTagName || !document.getElementById) return false;
    //测试id是否存在
    if (!document.getElementById("preview")) return false;

    var previews = document.getElementById("preview");
    var links = previews.getElementsByTagName("a");
    for (var i=0; i<links.length; i++)
    {
        links[i].onclick = function()
        {
            changePic(this);
            return false;
        }
    }
}
```
HTML文档作如下更改
```
<ul id="preview">
    <li>
        <a href="pre" class="preview" >上一个</a>
    </li>
    <li>
        <a href="next" class="preview" >下一个</a>
    </li>
</ul>
```
在原来基础的popup函数添加一个原图查看功能，能够读出当前图片的URL并在新窗口打开
HTML文档不需要修改，修改一下prepareLinks函数
```
function popUp (winURL)
{
    if (!document.getElementsByTagName || !document.getElementById) return false;
    if (!document.getElementById("picture")) return false;    
    var previewpic = document.getElementById("picture");
    var picURL = previewpic.getAttribute("src")
    window.open(picURL, "popup", "width=320,height=480");
}
```

# 共享onload事件
上面两个脚本都使用了window.onload事件，但是后面的赋值会把之前的覆盖掉，像下面这样会出问题
```
window.onload = prepareLinks;
window.onload = preparePreview;
```
有一种方法可以避过这个难题，同时容纳两个函数
```
window.onload = function()
{
    prepareLinks();
    preparePreview();
}
```
当需要绑定的函数不太多的时候，这个方法是可行简单的
还有一个比较有弹性的解决方案，编写一个addLoadEvent函数，实现功能如下

+ 把现有的window.onload事件处理函数的值存入变量oldonload
+ 如果没有绑定任何函数，只需要把新函数添加给它
+ 如果在这个处理函数上已经绑定了一些函数，就把新函数追加到现有指令的末尾

实现的函数如下
```
function addLoadEvent (func)
{
    var oldonload = window.onload;
    if (typeof window.onload != 'function')
    {
        window.onload = func;
    }
    else
    {
        window.onload = function()
        {
            oldonload();
            func();
        }
    }
}
```
唯一参数是函数名称，使用时直接调用
```
addLoadEvent(prepareLinks);
addLoadEvent(preparePreview);
```
使用三元运算符替代之前的return false，当成功执行时返回false，否则返回true
如果返回true，点击链接之后的动作正常进行，不会被JavaScript打断
```
return popUp(this.href) ? false : true;
return changePic(this) ? false : true;
```

# 键盘访问
当用户使用回车键而不是鼠标点击时，应该执行相同的效果，所以把onclick的值直接赋值给onkeypress
```
links[i].onkeypress = links[i].onclick
```
但这样处理并不友好，当用户按下任意按键，包括tab键都会触发这个事件函数，所以还是暂时不要使用这种处理方式
其实，使用tab键移动到链接的地方，按下enter仍然能触发onclick事件函数

下面使用CSS美化一下界面，给需要美化的标签元素添加class属性
```
<div id="preview">
    <table >
        <tr>
            <td class="change"><a href="pre" >上一个</a>
            <td class="change"><a href="next" >下一个</a>
        </tr>
    </table>
</div>
<div class="message"><a href="http://www.baidu.com" class="popup">查看原图</a></div>
<p id="description" class="message">picture-0</p>
```
然后编写简单的CSS
```
.change
{
    width: 400px;
}
.message
{
    TEXT-ALIGN: center;
    color: green;
}
```

# DOM Core和HTML-DOM
已经使用过的DOM方法如下

+ getElementById
+ getElementsByTagName
+ getElementsByClassName
+ getAttribute
+ setAttribute

这些方法都是DOM Core的组成部分，他们并不专属JavaScript，支持DOM的任何程序设计语言都可以使用他们，可以用来处理任何一种标记语言编写出来的文档

在使用JavaScript语言和DOM为HTML文件编写脚本时，还有许多属性可供选用
例如onclick，用于事件管理，这些属于HTML-DOM，在DOM Core出现之前就已经为人所知

比如说，HTML-DOM提供一个forms对象，这个对象可以把下面的语句
```
document.getElementsByTagName("form")
```
简化成
```
document.forms
```
类似的，HTML-DOM还提供了许多描述各种HTML元素的属性，比如
```
element.src
```
可以看到HTML-DOM通常会更短，但是他只能处理Web文档
