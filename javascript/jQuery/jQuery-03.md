---
title: jQuery-03 DOM操作
date: 2016-05-02 12:50:29
tags: [jquery]
---

# jQuery 中的DOM操作
## 查找节点
### 查找元素节点
获取元素节点并打印出文本内容
```
$(document).ready(function() //等待DOM加载完成
{
    var $li = $("ul li:eq(1)"); //ul下index为1的li子元素
    alert($li.text()); //打印文本
})
```
### 查找属性节点
`attr()`设置或者返回元素的属性值
```
$(document).ready(function() //等待DOM加载完成
{
    var $li = $("#logop > img");
    alert($li.attr("src")); //返回属性src值
    $li.attr("src","./static/pic2/pic0.jpg"); //设置src值
    $li.attr({"src":"./static/pic2/pic0.jpg","alt":"image"}); //设置src值和alt值
    $li.attr("alt", function(){return this.src}); //设置alt为其src值
})
```
## 创建节点
例如在ul元素下再插入两个li子元素，需要如下步骤

1. 创建两个li元素
2. 将两个新元素插入文档

```
$(document).ready(function() //等待DOM加载完成
{
    //直接使用工厂函数创建标签，元素节点、文本节点和属性节点都已添加
    var $li1 = $("<li>11 <a href='#'>南家三姐妹</a></li>");
    var $li2 = $("<li>12 <a href='#'>天使心跳</a></li>");

    //使用append()添加到文档节点
    var $ul = $(".dongmanRange  ul");
    $ul.append($li1);
    $ul.append($li2);
})
```
## 插入节点
这些方法不仅能实现插入，还能实现元素的移动

方法 | 描述
-------|-------
append() | 追加
appendTo() |  追加到
prepend() | 前置内容
prependTo() | 前置内容到
after() | 元素后追加
before() | 元素前追加
insertAfter() | 元素后追加
insertBefore() | 元素前追加

## 删除节点
如果文档中某个元素多余，可以删除这个元素，jQuery提供了两种删除节点的方法

### `remove()`方法
根据内容移除标签
```
$(document).ready(function()
{
    var $li = $(".dongmanRange > ul > li");
    $li.remove(":contains('天使心跳'),:contains('火影')")
})
```
### `empty()`方法
清空某个元素
```
$(document).ready(function()
{
    var $ul = $(".dongmanRange > ul");
    $ul.empty();
})
```
## 复制节点
当点击一个链接时，克隆一个自己
```
$(document).ready(function()
{
    var $li = $(".dongmanRange > ul >li");
    var $ul = $(".dongmanRange > ul");
    $li.click(function()
    {
        $(this).clone().appendTo($ul);
        return false;
    })
})
```
## 替换节点
如果要替换某个节点，使用`replaceWith()`和`replaceAll()`
```
$(document).ready(function()
{
    var $li = $(".dongmanRange > ul >li");
    var $ul = $(".dongmanRange > ul");
    $li.click(function()
    {
        $(this).replaceWith("<li><a href='#'>-</a></li>"); //点击之后，链接变成“-”
        return false;
    })
})
```
点击一次之后，所有链接都被替换
```
$(document).ready(function()
{
    var $li = $(".dongmanRange > ul >li");
    var $ul = $(".dongmanRange > ul");
    $li.click(function()
    {
        $(this).replaceAll($li);
        return false;
    })
})
```
## 包裹节点
如果要将某个节点用其他标记包裹起来，jQuery提供了相应的方法
### `wrap()` 包裹
```
$(document).ready(function()
{
    $('a').wrap("<li class='promoted'></li>"); //给每个a元素包裹一个li元素
})
```
### `unwrap()` 去除一层包裹
```
$(document).ready(function()
{
    var $li = $(".dongmanRange > ul >li");
    $li.children('a').unwrap().wrap("<li class='promoted'></li>"); //取出原本包裹，添加新包裹
})
```
### `wrapAll()` 包裹所有
```
$(document).ready(function()
{
    $("p").wrapAll("<div></div>"); //用div包裹所有p元素
})
```
### `wrapInner()` 方法
将每个匹配的元素的子内容用其他结构包起来
HTML
```
<div class="test">
    hello
</div>
<div class="test">
    world
</div>
```
JavaScript
```
$('.test').wrapInner("<p></p>");
```
结果
```
<div class="test">
    <p>hello</p>
</div>
<div class="test">
    <p>world</p>
</div>
```

## 属性操作
attr()可以读取和设置属性，使用removeAttr()可以删除属性
```
$(document).ready(function()
{
    var $li = $(".dongmanRange > ul >li");
    $li.removeAttr("title");
})
```
## 样式操作
### 获取和设置样式
可以直接使用`attr()`操作calss属性

### 追加样式
jQuery提供专门的`addClass()`方法来追加样式

+ 如果给一个元素添加了多个样式，那么就相当于合并了他们的样式
+ 如果有不同的class设定了同一样式，后者覆盖前者

### 移除样式
与`addClass()`相反，使用`removeClass()`来移除样式

### 判断是否含有某个样式
`hasClass()`可以判断元素中是否含有某个class，如果有，返回true

## 设置和获取HTML、文本和值
### `html()` 方法
类似于JavaScript的`innerHTML`属性，可以用来读取或者设置某个元素的HTML内容
```
$(document).ready(function()
{
    var $li = $(".dongmanRange > ul >li");
    $li.html("new") //所有列表的内容都变成“new”了
})
```
### `text()` 方法
类似于JavaScript的`innerText`属性，读取或设置元素的文本内容不含标签
```
$(document).ready(function()
{
    var $li = $(".dongmanRange > ul >li");
    $li.text("aa");
})
```
### `val()` 方法
类似于JavaScript的`value`属性，可以用来设置和获取元素的值，无论是文本框、下拉列表还是单选框，都会返回元素值

在图片浏览的网页添加一个跳页的表单，然后点击submit之后跳页

这里使用了`focus()`方法和`blur`方法，当鼠标焦点没有选中时显示"页码"，当选中时清空等待用户输入
使用`val()`方法获得用户输入的页码的数值，然后跳页
```
$(document).ready(function()
{
    //focus()方法相当于onfocuse()方法，处理当获得焦点时事件
    $("#pageJump").focus(function() 
    {
        var textvalue = $(this).val();
        if (textvalue == "页码")
        {
            $(this).val("");
        }
    });

    //blur()相当于onblur()方法，处理当失去焦点时事件
    $("#pageJump").blur(function() 
    {
        var textvalue = $(this).val();
        if (textvalue == '')
        {
            $(this).val("页码");
        }
    });

    $("#submit").click(function()
    {
        var txtPage = $('#pageJump').val();
        if (txtPage == '')
        {
            alert("请输入页码");
        }
        else
        {
            pageNum = parseInt(txtPage);
            if (pageNum >= 0)
            {
                num = pageNum
                source = "./static/pic2/pic" + pageNum + ".jpg";
                $("#description").html("<em>picture-</em> "+ pageNum);
                $("#picture").attr("src", source);
                return false;
            }
            else
            {
                alert("输入的页码无效")
            }
        }
    });
})
```
`val()` 方法也可以设置`select(下拉列表框)`，`checkbox(多选框)`和`radio(单选框)`相应的被选中的值
```
$(":checkbox").val(["a","b"]); //多选框‘
$(":radio").val(["c"]); //单选框
```
## 遍历节点
### `children()` 方法
用于匹配元素的子元素集合，只考虑子元素而不考虑其他后代元素

### `next()` 方法
用于匹配元素后面一个同辈元素

### `siblings()` 方法
用于匹配元素所有同辈元素

### `closest()` 方法
寻找最近的元素

寻找最近的`li`元素，并设置颜色为红色
```
$("#main").closest("li").css("color","red");
```

除此之外，jQuery还有许多哦遍历节点的方法，比如`find()`、`filter()`、`nextAll()`、`prevAll()`、`parent()`和`parents()`等

## CSS-DOM 操作
CSS-DOM操作简单来说就是读取和设置style对象的各种属性，但是无法通过它提取通过外部设置的样式属性

在jQuery中，可以直接利用`css()`方法提取元素的样式属性
```
$(document).ready(function()
{
    alert($(".message").css("color")); //获取class为message的元素的颜色
})
```
可以直接使用`css()`方法设置某个元素的样式
```
$(document).ready(function()
{
    $(".message").css("color","red");//设置class为message的元素的颜色为红色
})
```
和`attr()`方法一样，`css()`方法也可以同时设置多个样式属性
```
$(document).ready(function()
{
    $(".message").css({"color":"red","backgroundColor":"green"});
})
```
常用方法

### `offset()` 方法
获取元素在当前视窗的相对偏移
```
$(document).ready(function()
{
    var offset = $("#picture").offset(); 
    var left = offset.left; //左偏移
    var top = offset.top; //右偏移
})
```

### `position()` 方法
获取元素相对父元素的偏移，同样，分为左右偏移
```
$(document).ready(function()
{
    var offset = $("#picture").position();
    var left = offset.left;
    var top = offset.top;
})
```

### `scrollTop()` 方法和`srollLeft()` 方法
获取元素相对滚动条的偏移

## 实例
跟踪鼠标的超链接提示和图片预览功能

```
$(document).ready(function() //等待DOM加载完毕
{
    $(".linkp a img").mouseover(function(e) //鼠标移动到链接上触发
    {
        this.myTitle = this.title; //取出title值
        this.title = ""; //清空原来的title值，这样就不会触发浏览器自带的提示

        var $tip = $("<div id='tip'>"+this.myTitle+"</div>"); //创建一个显示title的div元素
        $("#main").append($tip); //追加到class为main的元素里面
        $tip.css(   //获得鼠标位置并稍微错位方便观察
        {
            "top": (e.pageY+10) + "px", 
            "left": (e.pageX+20) + "px"
        }).show(); //显示
    })
    .mouseout(function() //鼠标移开连接触发
    {
        this.title = this.myTitle; //还原清空的title值
        $("#tip").remove(); //删除提示元素
    })
})
```
添加图片预览功能

```
$(document).ready(function() //等待DOM加载完毕
{
    $(".linkp a img").mouseover(function(e) //鼠标移动到链接上触发
    {
        this.myTitle = this.title; //取出title值
        this.title = ""; //清空原来的title值，这样就不会触发浏览器自带的提示
        var tiptitle = "<br / ><div>"+this.myTitle+"</div>"; //提示元素
        var $tippic = $("<div id='tip'><img src='"+this.src+"' alt='预览' / >"+tiptitle+"</div>"); /预览图片元素
        $("#main").append($tippic); //追加到class为main的元素上
        $tippic.css(  //获得鼠标位置并稍微错位方便观察
        {
            "top": (e.pageY-200) + "px",
            "left": (e.pageX-200) + "px"
        }).show();
    })
    .mouseout(function() //鼠标移开连接触发
    {
        this.title = this.myTitle; //还原清空的title值
        $("#tippic").remove(); //删除提示元素
    })
    .mousemove(function(e) //鼠标移动时触发
    {
        $("#tip").css(
        {
            "top": (e.pageY-200) + "px",
            "left": (e.pageX-200) + "px"
        });
    })
})
```
