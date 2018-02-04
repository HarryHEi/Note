---
title: javascript-07
date: 2016-04-29 15:59:26
tags: [javascript]
---

# 显示缩略语列表
使用JavaScript把abbr标签的title属性集中在一起显示，创建的显示列表如下图的格式
```
<dl>
    <dt>A</dt>
    <dd>AAAAA</dd>
    <dt>B</dt>
    <dd>BBBBB</dd>
    ...
</dl>
```
具体步骤如下

+ 遍历这份文档中的所有abbr元素
+ 保存每个abbr元素的title属性
+ 保存每个abbr元素包含的文本
+ 创建一个列表元素(dl)
+ 遍历刚才保存的title属性和abbr元素的文本
+ 创建一个标题元素(dt)
+ 创建一个描述元素(dd)
+ 把abbr文本属性插入到标题元素(dt)
+ 把title属性插入到描述元素(dd)
+ 把dt和dd追加到dl上
+ 把dl追加到body上

# 编写displayAbbr函数
```
function displayAbbr()
{
    //取得所有abbr标签
    var abbrs = document.getElementsByTagName("abbr");
    //检测有没有abbr标签
    if (abbrs.length < 1) return false;
    var defs = new Array();
    //遍历读取title属性和标签内容，标签内容也可以使用lastChile.nodeValue
    for (var i=0; i<abbrs.length; i++)
    {
        var defi = abbrs[i].getAttribute("title");
        var key = abbrs[i].innerHTML;
        defs[key] = defi;
    }
    //父标签
    var artic = document.getElementById("articals")
    var dl = document.createElement("dl");
    artic.appendChild(dl);
    //循环插入子标签
    for (key in defs)
    {
        var dt = document.createElement("dt");
        var dd = document.createElement("dd");
        dl.appendChild(dt);
        dl.appendChild(dd);
        dt.innerHTML = key;
        dd.innerHTML = defs[key];
    }
}
```
由于IE浏览器不支持abbr标签，拒绝承认abbr为元素标签，所以必须考虑到兼容问题
```
function displayAbbr()
{
    var abbrs = document.getElementsByTagName("abbr");
    if (abbrs.length < 1) return false;
    var defs = new Array();
    for (var i=0; i<abbrs.length; i++)
    {
        //如果检测不到子元素，进入下一次循环
        if (abbrs[i].childNodes.length<1) continue;
        var defi = abbrs[i].getAttribute("title");
        var key = abbrs[i].innerHTML;
        defs[key] = defi;
    }
    var artic = document.getElementById("articals")
    var dl = document.createElement("dl");
    for (key in defs)
    {
        var dt = document.createElement("dt");
        var dd = document.createElement("dd");
        dl.appendChild(dt);
        dl.appendChild(dd);
        dt.innerHTML = key;
        dd.innerHTML = defs[key];
    }
    //如果内容数为0，返回flase
    if (dl.childNodes.length<1) return false;
    artic.appendChild(dl);
}
```

# 操作cookie

使用DOM可以获得或者更改cookie
```
alert(document.cookie)
document.cookie="uuid=12345"
```
