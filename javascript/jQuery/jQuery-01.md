---
title: jQuery-01 简述
date: 2016-05-01 10:48:01
tags: [jquery]
---

# 简述
JavaScript是网景公司开发的脚本语言，jQuery是JavaScript开发的一个轻量级库
现在正在阅读《锋利的jQuery》,此书一共八章，这篇博客是阅读过程中的笔记

[代码](https://github.com/HarryHEi/Guidence.git)

# 中文文档
http://jquery.cuishifeng.cn/

# 安装
在head标签引入
```
<script type="text/javascript" src="./static/js/jquery-1.12.3.js"></script>
```

# 编写简单的jQuery代码
在jQuery中`$`就是jQuery的简写形式，例如`$("#foo")`和`jQuery("#foo")`是等价的，下面编写第一个jQuery程序
```
$(document).ready
(
    function()
    {
        alert("hello world");
    }
)
```
可以简写
```
$(
    function()
    {
        alert("hello world");
    }  
)
```
这段代码类似于JavaScript中的window.onload方法，不过相比`window.onload`还是有些区别

对比 | window.onload | $(document).ready()
-------|------------------------|----------------------------
执行时机 | 必须等待网页加载完毕之后(包括图片)才能执行 | 网页中所有DOM结构绘制完毕后就执行
编写个数 | 不能同时编写多个 | 能够同时编写多个
简化写法 | 无 | $(function(){...})

# 代码风格
## 链式操作风格
创建一个导航栏文档
```
<div id="menu">
    <div class="has_children">
        <span >看图</span>
        <a >二次元</a>
        <a >三次元</a>
    </div>
    <div class="has_children">
        <span >听歌</span>
        <a >二次元</a>
        <a >三次元</a>
    </div>
    <div class="has_children">
        <span >看片</span>
        <a >二次元</a>
        <a >三次元</a>
    </div>
</div>
```
CSS内容如下，如要是添加了高亮类和隐藏标签
```
#menu
{
	width: 300px;
	padding: 50px;
}
.has_children
{
	color: black;
	cursor: pointer;
	font-size: 20px;
}
.highlight
{
	color: #3299cc;
}
.has_children > a
{
	display: none;
	float: left;
	width: 300px;
	height: 20px;
	font-size: 16px;
	padding-left: 20px;
}
```
## 项目举例
例如，项目需求是制作导航栏，单机不同名称链接，显示相应内容
代码内容是当点击到class中含有has_children的元素时，给其添加一个highlight类属性，然后显示出它的所有a元素子类
而同辈元素都移除highlight属性并隐藏a子类，这就是链式操作，理论上就是一行代码
```
$(document).ready(function() //等待DOM加载完毕
{ 
    $(".has_children").click(function()
    {
        $(this).addClass("highlight") //添加highlight类
            .children("a").show().end() //显示a子类
            .siblings().removeClass("highlight") //删除同辈highlight类
            .children("a").hide(); //隐藏同辈a子类
    })
})
```
# jQuery对象和DOM对象
## DOM对象
DOM是文档对象模型的缩写，每一分DOM对象都可以表示成一棵树

## jQuery对象
jQuery对象就是通过jQuery包装DOM对象后产生的对象
如果一个对象是jQuery对象，那么就可以使用jQuery里的方法 

例如
```
$(document).ready(function()
{
    alert($("#main").html());
})
```
这段代码等同于
```
$(document).ready(function()
{
    alert(document.getElementById("main").innerHTML)
})
```
jQuery对象无法使用DOM对象的方法，例如$('#main').innerHTML是不合法的

## 相互转换
### jQuery对象转化成DOM对象
jQuery不能使用DOM方法，如果不得不使用DOM对象时，有以下两种处理方法

`[index]`
jQuery对象是一个数组对象，可以通过[index]的方法得到响应的DOM对象
```
$(document).ready(function()
{
    var $menu = $("#menu");
    var menu = $menu[0];
    alert(menu.innerHTML);
})
```

`get(index)`
这个方法是jQuery本身提供的，通过`get(index)`方法得到相应的DOM对象
```
$(document).ready(function()
{
    var $menu = $("#menu");
    var menu = $menu.get(0);
    alert(menu.innerHTML);
})
```
### DOM对象转化成jQuery对象
对于DOM对象，只需要用`$()`把DOM对象包起来，就获得了一个jQuery对象，即`$(DOM对象)`
```
$(document).ready(function()
{
    var memu = document.getElementById("memu");
    var $menu = $(menu);
    alert($menu.html());
})
```
## 实例研究
实现一个“同意并接受条款”复选框，只有选中才能提交

文档中添加一个复选框，然后使用DOM方法判断是否选中
```
$(document).ready(function()
{
    var $box = $("#box");
    var box = $box[0];
    $box.click(function()
    {
        if (box.checked) //使用DOM方法判断是否选中
        {
            alert("感谢支持!")
        }
    })
})
```
也可以使用jQuery方法判断是否选中
```
if ($box.is(":checked")) #...
```
# jQuery和其他库的冲突
在jQuery库中，几乎所有的插件都被限制在它的命名空间，通常，全局对象都被很好地储藏在jQuery命名空间，因此和其他库不会引起冲突
## 在其他库之后导入
在其他库加载完毕后，使用jQuery.noConflick函数来将变量`$`的控制权交给其他库
```
jQuery.noConflict(); //将变量$控制权交给其他库
```
如果确保jQuery不会与其他库冲突，并自定义快捷方式，可以像这样
```
var $j = jQuery.noConflict(); //自定义一个快捷方式

$j(document).ready(function()
{
    #...
})

$("pp").style.display = "none"; //使用其他库
```
如果不想自定义备用名称，而继续使用$，不管其他库的$()方法，可以使用下面两种方法

方法1
```
jQuery.noConflict(); //将变量$控制权交给其他库

jQuery(function($) //设定页面加载时函数
{
    $(".has_children").click(function() //在函数内部继续调用$()方法
    {
        $(this).addClass("highlight")
            .children("a").show().end()
            .siblings().removeClass("highlight")
            .children("a").hide();
    })
})

$("pp").style.display = "none"; //使用其他库
```
方法2
```
jQuery.noConflict(); //将变量$控制权交给其他库

(function($) //定义匿名函数并设置形参为$
{
    $(document).ready(function() //等待DOM加载完毕
    { 
        $(".has_children").click(function()
        {
            $(this).addClass("highlight") //添加highlight类
                .children("a").show().end() //显示a子类
                .siblings().removeClass("highlight") //删除同辈highlight类
                .children("a").hide(); //隐藏同辈a子类
        })
    })
})(jQuery); //执行匿名函数并传递实参jQuery

$("pp").style.display = "none"; //使用其他库
```
## 在其他库之前导入
如果jQuery库在其他库之前就导入了，那么可以直接使用jQuery来做一些jQuery的工作
同时，可以使用$()方法作为其他库的快捷方式
```
jQuery(function() //直接使用jQuery
{
    $(".has_children").click(function() //在函数内部继续调用$()方法
    {
        $(this).addClass("highlight")
            .children("a").show().end()
            .siblings().removeClass("highlight")
            .children("a").hide();
    })
})

$("pp").style.display = "none"; //使用其他库
```
