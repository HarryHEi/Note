---
title: javascript-04
date: 2016-04-28 13:27:55
tags: [javascript]
---

# 打开一个新窗口
JavaScript使用window对象的open()方法创建新的浏览器窗口
window.open(url, name, features)
这三个参数都是可选的

+ 第一个参数是想要在新窗口打开的网页的URL地址，如果省略这个参数将打开一个空白的窗口
+ 第二个参数是新窗口的名字，在代码里可以通过这个名字与新窗口通信
+ 最后一个参数是以逗号分隔的字符串，内容是新窗口的各种属性，包括尺寸以及各种功能

open()方法是使用BOM(窗口对象模型)的一个很好的案例，它对文档的内容无任何影响(DOM的地盘)，只与浏览环境有关
下面是window.open()的典型应用，这个函数将打开一个320像素宽，480像素高的新窗口"popup"
所以当再次调用函数推送一个新URL时，不会再创建一个新窗口，而是直接在这个名为popup的窗口替换文档
```
function popUp (winURL)
{
    window.open(winURL, "popup", "width=320,height=480");
}
```
调用popUp函数的一个办法是使用伪协议(pseudo-protocol)

# "javascript:" 伪协议
真协议用来在因特网上的计算机之间传输数据包，如HTTP、FTP协议等，伪协议是一种非标准化协议
伪协议让我们通过一个链接来调用JavaScript函数

下面是通过"javascript:"伪协议调用popUp()函数的具体做法
```
<a href="javascript:popUp('http://www.baidu.com');">baidu</a>
```
但是，如果浏览器不支持这种伪协议或者禁用了JavaScript功能，什么也不会做

# 内嵌的事件处理函数
之前在图片预览脚本中使用的onclick事件处理函数就是嵌入到标签
同样，可以使用这个技巧调用popUp函数，href实际上已经没有作用，所以直接这样写
```
<a href="#" onclick="popUp('http://www.baidu.com'); return false;">baidu</a>
```
嵌入事件函数和之前的伪协议相同，如果浏览器禁用了JavaScript功能，这个链接不起作用

如果希望能够兼容那些没有JavaScript功能的用户比如搜索机器人，可以把href改为真正的链接
```
<a href="http://www.baidu.com" onclick="popUp(this.getAttribute('href')); return false;">baidu</a>
```
其中this.getAttribute('href')可以简写成this.href
```
<a href="http://www.baidu.com" onclick="popUp(this.href); return false;">baidu</a>
```

# 分离 JavaScript
之前的代码已经和html文档分的很开了，但是问题出在内嵌的事件处理函数
可以把一个事件添加到某个带有特定id属性的元素上
```
getElementById(id).event = action
```
如果涉及多个元素可以使用getElementsByTagName、getElementsByClassName和getAttribute把事件添加到有特定属性的元素
具体步骤如下：

1. 把文档里的所有链接放入一个数组
2. 遍历数组
3. 如果某个链接的class属性为popup，就表示这个链接在被点击时应该调用popUp函数

在html文档里把元素的class属性设为popup
```
<a href="http://www.baidu.com" class="popup">baidu</a>
```
在js脚本里给class属性为popup的元素添加功能
```
var links = document.getElementsByTagName("a");
for (var i=0; i<links.length; i++)
{
    if (links[i].getAttribute("class") == "popup")
    {
        links[i].onclick = function()
        {
            popUp(this.href);
            return false;
        }
    }
}
```
有一点要注意，这个js脚本代码如果是在html开头部分导入的，那么一开始将无法执行这条语句，因为文档还没有被加载
```
var links = document.getElementsByTagName("a");
```
所以必须要在文档加载完毕之后再加载这个脚本，使用window.onload时间函数可以做到这点
```
function prepareLinks ()
{
    var links = document.getElementsByTagName("a");
    for (var i=0; i<links.length; i++)
    {
        if (links[i].getAttribute("class") == "popup")
        {
            links[i].onclick = function()
            {
                popUp(this.href);
                return false;
            }
        }
    }
}
window.onload = prepareLinks;
```

# 对象检测
如果用户浏览器版本比较老，可能无法支持一部分JavaScript语句，所以需要在脚本里添加一个检测语句
```
if (!document.getElementsByTagName || !document.getElementById) return false;
```
添加到之前写的脚本函数当中
```
function prepareLinks ()
{
    if (!document.getElementsByTagName) return false;
    var links = document.getElementsByTagName("a");
    for (var i=0; i<links.length; i++)
    {
        if (links[i].getAttribute("class") == "popup")
        {
            links[i].onclick = function()
            {
                popUp(this.href);
                return false;
            }
        }
    }
}
window.onload = prepareLinks;
```

# 性能考虑
## 尽量少访问DOM和尽量减少标记
访问DOM的方式会对脚本性能产生非常大的影响，灵活使用变量

尽量减少文档中标记的数量，过多不必要的元素会增加DOM树的规模，增加遍历DOM树的时间

## 合并和放置脚本
推荐做法是把所有js脚本合并到一个文件当中，减少页面加载时发送请求的数量

一般来说，更具HTTP规范，浏览器每次从同一个域名中最多同时下载两个文件，在下载期间，浏览器不会下载其他任何文件
把所有的scrip标签都放到文档末尾，/body标签之前，就能让页面加载更快

## 压缩脚本
所谓压缩脚本，就是把脚本文件中的不必要的字节，比如空格和注释，全部删除，从而达到压缩文件的目的
好在，很多工具都可以做这件事，有的工具甚至会重写你的代码，使用更短的变量名

精简后的代码虽然不容易看懂，但是能大幅减少文件的大小，多数情况下应该有两个版本，一个是工作副本，另一个是精简副本

通常，为了区分，最好在精简副本的文件名加上min字样
```
<script src="./static/js/test.min.js"></script>
```
