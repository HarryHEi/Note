---
title: javascript-08 style属性
date: 2016-04-30 15:14:39
tags: [javascript]
---

# CSS-DOM
## 三位一体的网页
网页由三层信息构成

+ 结构成
+ 表示层
+ 行为层

### 结构层
网页的结构层由HTML或者XHTML之类的标记语言创建，标签对网页内容的含义作出了描述
但是这些标签不包含任何关于内容如何显示的信息

### 表示层
表示层由CSS负责完成，CSS描述网页内容如何呈现

### 行为层
行为层负责内容应该如何响应事件这一问题

## style 属性
每个元素都有意个style属性，包含着元素的样式，查询这个属性将返回一个对象，样式都放在这个style对象的属性里
### 获取样式
例如获取color属性
```
addLoadEvent(
    function()
    {
        var example = document.getElementById("message");
        alert(example.style.color);
    }
)
```
当属性有“-“号时，要使用驼峰命名法
```
example.style.fontFamily
```
DOM style属性不能用来检索外部CSS文件里声明的样式

### 设置样式
style对象的各个属性都是可读可写的，可以使用赋值操作来更新样式
```
addLoadEvent(
    function()
    {
        var example = document.getElementById("message");
        example.style.color="blue";
    }
)
```
## 如何应用DOM脚本设置样式
### 根据元素在节点树的位置设置样式
CSS无法根据元素之间的相对位置找出特定元素，但是对DOM来说并不难
可以根据元素的位置声明样式

### 更具某种条件反复设置样式
JavaScript擅长处理重复性任务，比如给一个表格添加样式

### 响应事件
比如设置当鼠标悬停时，文本改变颜色
```
addLoadEvent(
    function highLight()
    {
        var message = document.getElementById("message");
        message.onmouseover = function()
        {
            this.style.color="red";
        }
        message.onmouseout = function()
        {
            this.style.color="#70DDdd";
        }
    }
)
```
## className 属性
如果直接使用脚本更改style属性，后期的修改会比较麻烦，更好的方法是修改className属性
可以使用setAttribute做到这件事
```
example.setAttribute("class", "examp")
```
更简单的方式是修改className属性
```
example.className = "examp"
```
修改完className后，只需要在CSS文件中添加样式就行了，如果给一个元素追加新样式，步骤如下

+ 检查className属性是否为空
+ 如果是，把class设置值直接赋值给className属性
+ 如果不是，把一个空格和新的class设置值追加到className属性上(example.className += " examp")
