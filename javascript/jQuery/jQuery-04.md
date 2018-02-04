---
title: jQuery-04 jQuery中的事件和动画
date: 2016-05-03 09:06:47
tags: [jquery]
---

jQuery增加并扩展了基本的事件处理机制

# jQuery中的事件
## 加载DOM
在页面加载完毕之后，浏览器会通过JavaScript为DOM元素添加事件

在常规JavaScript中，通常使用window.onload方法
在jQuery中，使用`$(document).ready()`方法

### 执行时机
window.onload方法是在网页中所有元素完全加载到浏览器之后才执行，包括文档和图片

jQuery中的`$(document).ready()`在DOM完全就绪时就可以被调用，这是并不意味者所有文件和图片已经加载完毕

如果处理对象绑定在window上，则会在所有内容加载完执行

### 简写形式
```
$(document).ready(function()
{
	//code
})
```
可以简写成
```
$(function()
{
	//code
})
```
`$(document)`可以简写成`$()`，默认参数就是document
```
$().ready(function()
{
	//code
})
```
## 事件绑定
在文档装载完毕之后，如果打算为元素绑定事件来完成某项操作，可以使用bind()方法

第一个参数是含有一个或多喝事件类型的字符串，以空格隔开
第二个参数是可选参数，作为event.data属性值传递给事件对象的额外数据对象
第三个参数绑定处理函数，如果值为false会使默认动作失效

### 基本效果
之前的目录可以这样写
```
$(function()
{ 
    $(".has_children").bind('mouseover click',function()
    {
        $(this).addClass("highlight")
            .children("a").show().end()
            .siblings().removeClass("highlight")
            .children("a").hide();
        return false; //通过false取消默认行为
    })
})
```
### 简写
对于click、mouseover和mouseout这类事件，可以使用简写方式
```
$(function()
{ 
    $(".has_children").mouseover(function()
    {
        $(this).addClass("highlight")
            .children("a").show().end()
            .siblings().removeClass("highlight")
            .children("a").hide();
    })
})
```
## 合成事件
jQuery有两个合成事件，`hover()`方法和`toggle()`方法，`toggle()`方法在1.9版本移除

`hover()`方法的语法结构
```
houver([over,] out)
```
第一个参数是鼠标移动到元素上要触发的函数
第二个参数是移除元素要触发的函数

当鼠标移动到元素上停止不动时，不会再触发任何函数
```
$(function()
{
    $("#message").hover(function()
    {
        $(this).hide();
    },
    function()
    {
        $(this).show();
    })
})
```
## 事件冒泡
当多个元素被绑定同一事件，且元素为嵌套关系，当触发事件时，事件会按照DOM层次结构像水泡一样不断向上直至顶端

### 事件对象
获取事件对象只需要添加一个参数
```
$(function()
{ 
    $(".has_children").bind('mouseover',function(event) //event:事件对象
    {
        //
    })
})
```
### 停止冒泡事件
使用`stopPropagation()`方法停止冒泡事件
文档结构如下
```
<div id="test1">
    <p id="test2">
        <a id="test3" href="#">
            <img src="./static/image/logo.png" id="test4" / >
        </a>
    </p>
</div>
```
下例只输出4
```
$(function()
{
    $("#test1").click(function()
    {
        alert("1")
    }),
    $("#test2").click(function()
    {
        alert("2")
    }),
    $("#test3").click(function()
    {
        alert("3")
    }),
    $("#test4").click(function(event)
    {
        alert("4")
        event.stopPropagation();
    })
})
```
下例输出4、3、2
```
$(function()
{
    $("#test1").click(function()
    {
        alert("1")
    }),
    $("#test2").click(function(event)
    {
        alert("2")
        event.stopPropagation();
    }),
    $("#test3").click(function()
    {
        alert("3")
    }),
    $("#test4").click(function()
    {
        alert("4")
    })
})
```
也可以使用
```
return false;
```
### 阻止默认行为
通过preventDefault()只取消默认行为
```
$(function()
{ 
    $(".has_children").bind('click',function(event)
    {
        //...
        event.preventDefault();
    })
})
```
也可以使用
```
return false;
```
## 事件对象的属性
jQuery对事件对象的常用属性进行了封装
### `event.type` 属性
获得事件类型
```
$("#test4").click(function(e)
{
    alert(e.type)
    return false;
})
```
### `event.preventDefault()` 方法
阻止默认事件

### `event.stopPropagation()` 方法
阻止冒泡事件

### `event.target` 属性
获取触发事件的DOM元素
```
$(function()
{
    $("#test4").click(function(e)
    {
        alert(e.target.src)
        return false;
    })
})
```
### `event.relatedTarget` 属性
事件中涉及的其他DOM元素
```
$("#test4").mouseout(function(event) 
{
    alert(event.relatedTarget.nodeName); // "DIV"
});  
```
### `event.pageX/pageY` 属性
event.pageX 鼠标相对于文档的左边缘的位置
event.pageY 鼠标相对于文档顶部边缘的位置

### `event.which` 属性
针对鼠标和键盘事件，这个属性能确定按的哪个键
```
$(function()
{
    $("#test4").click(function(e)
    {
        alert(e.which);
        return false;
    })
})
```
### `event.data` 属性
传递的数据
```
$("a").each(function(i) {
    $(this).bind('click', {index:i}, function(e){
       alert('my index is ' + e.data.index);
    });
});   
```
## 移除事件
不仅可以为同一个元素绑定多个事件，也可以绑定多个同一事件

使用`unbind()`移除绑定事件

第一个参数是事件类型，第二个参数是要移除的函数(创建时需要把匿名函数赋值给一个变量)
如果没有参数，删除所有绑定的事件
如果只提供了事件类型，只删除该类型的绑定事件

## 模拟操作
使用`trigger()`方法可以用来触发一个事件
```
$("#test4").trigger("click")

$("#pageJump").trigger("focus")
```
也可以用来触发自定义事件
```
$("#test4").bind('myClick', function(e)
{
    alert("4");
    return false;
})
$("#test4").trigger("myClick")
```
# jQuery中的动画
## `show()`和`hide()`方法
如果调用`hide()`方法，会把一个元素的display样式改为"none"
```
$("element").hide()
```
相当于
```
$("element").css("display","none");
```
`show()`方法和`hide()`方法相反，会把display属性还原到调用`hide()`方法之前的状态

可以指定参数"slow"或者"fast"设置显示速度的快慢，也可以指定时间(ms)

## `fadeIn()`和`fadeOut()`方法
`fadeIn()` 和 `fadeOut()` 只改变元素的不透明度直至消失或出现，同样可以指定时间参数

## `slideUp()`和`slideDown()`方法
只会改变元素的高度
`slideUp()`是由下到上隐藏
`slideDown()`是由上到下出现

## 自定义动画方法`animate()`
`animate(params, speed, callback)`
第一个参数是样式属性和值的映射，比如`{top:"100px",left:"100px"}`
第二个参数是速度参数`("slow"，"normal"，or "fast")`，单位ms，可选
第三个参数是动画完成后执行的函数，可选

### 基本
通过点击两个链接，实现id为fly的元素的左右移动
```
$(function()
{
    $("#left").click(function()
    {
        $("#fly").animate({left:"200px"},"slow");
        return false;
    });

    $("#right").click(function()
    {
        $("#fly").animate({left:"600px"},"slow");
        return false;
    })
})
```

### 累加
```
$(function()
{
    $("#left").click(function()
    {
        $("#fly").animate({left:"-=50px"},"fast");
        return false;
    });

    $("#right").click(function()
    {
        $("#fly").animate({left:"+=50px"},"fast");
        return false;
    })
})
```

### 多重动画
在移动过程中同时改变透明度
```
$(function()
{
    $("#left").click(function()
    {
        $("#fly").animate({left:"-=50px",opacity:"toggle"},"slow");
        return false;
    });

    $("#right").click(function()
    {
        $("#fly").animate({left:"+=50px",opacity:"toggle"},"slow");
        return false;
    })
})
```
如果需要有先后顺序，可以链式赋值
```
$(function()
{
    $("#left").click(function()
    {
        $("#fly").animate({left:"-=50px"},"slow")
            .animate({height:"+=20px"},"slow");
        return false;
    });
})
```
### 回调函数
在动画执行完之后改变css样式
```
$(function()
{
    $("#left").click(function()
    {
        $("#fly").animate({left:"-=50px"},"slow", function()
            {
                $(this).css("src","./static/pic2/pic0.jpg");
            });
        return false;
    });
})
```
## 停止动画
`stop()`
第一个参数是停止动画的队列名称，可选
第二个参数如果设置为true，则清空队列，结束动画
第三个参数如果设置为true，完成队列，立即完成动画

调用`stop()`方法，会立即停止动画
```
$(function()
{
    $("#stop").click(function()
    {
        $("#fly").stop(); //停止动画
    });

    $("#right").click(function()
    {
        $("#fly").animate({left:"+=50px"},10000);
        return false;
    })
})
```
如果队列中有多个动画，摁一次只能停止一个动画，其他动画会继续执行
```
$(function()
{
    $("#stop").click(function()
    {
        $("#fly").stop();
    });

    $("#right").click(function()
    {
        $("#fly").animate({left:"+=50px"},5000)
            .animate({height:"+=50px"},5000); //链式动画
        return false;
    })
})
```
如果设置为true，会停止所有动画
```
$(function()
{
    $("#stop").click(function()
    {
        $("#fly").stop(true); //参数true
    });

    $("#right").click(function()
    {
        $("#fly").animate({left:"+=50px"},5000)
            .animate({height:"+=50px"},5000);
        return false;
    })
})
```
判断是否处于动画状态
```
$(function()
{
    $("#stop").click(function()
    {
        if ($("#fly").is(":animated"))
        {
            $("#fly").stop(true);
        }
    });
})
```
## 其他动画方法
### `slideToggle()`方法
通过高度变化来切换所有匹配元素的可见性，并在切换完成后可选地触发一个回调函数

点击之后会使元素在隐藏和显示两个状态切换
```
$(function()
{
    $("#right").click(function()
    {
        $("#fly").slideToggle("slow");
        return false;
    })
})
```
### `fadeToggle()`方法
通过不透明度的变化来开关所有匹配元素的淡入和淡出效果，并在动画完成后可选地触发一个回调函数。

点击之后会使元素在隐藏和显示两个状态切换
```
$(function()
{
    $("#right").click(function()
    {
        $("#fly").fadeToggle("slow");
        return false;
    })
})
```
### `delay()` 方法
设置一个延时来推迟执行队列中之后的项目。

```
$(function()
{
    $("#right").click(function()
    {
        $("#fly").animate({left:"+=50px"},"fast")
            .delay(6000) //设置延时
            .animate({height:"+=50px"},"fast");
        return false;
    })
})
```

# 实例
图片轮播效果

建立html文档
```
<div id="p_show">
    <div id="p_head">
        <a href="#" id="pre">《</a> <a href="#" id="next">》</a>
    </div>
    <div id="p_contant">
        <li><a href="https://www.bilibili.com/"><img src="./static/image/bilibili.jpg" alt="bilibili" title="Bilibili" /></a></li>
        <li><a href="https://www.zhihu.com/"><img src="./static/image/zhihu.jpg" alt="zhihu" title="知乎" /></a></li>
        <li><a href="https://www.python.com/"><img src="./static/image/python.jpg" alt="python" title="Python" /></a></li>
        <li><a href="https://www.baidu.com/"><img src="./static/image/baidu.jpg" alt="baidu" title="百度" /></a></li>
    </div>
    <div id="p_O">
        <span>。</span>
        <span>.   </span>
        <span>.   </span>
        <span>.   </span>
    </div>
</div>
```
CSS样式
```
#p_head
{
    position: absolute;
    left: 700px;
}
#p_head a 
{
    background-color: #fff;
    text-decoration: none;
}
#p_contant li
{
    position: absolute;
    left: 700px;
    top: 150px;
    list-style:none;
}
#p_O
{
    position: absolute;
    left: 700px;
    top: 350px;
    font-size: 20px;
}
```

```
var page=0 //储存当前显示的图片index

$(function() //等待DON加载完成
{
    $("#p_contant > li:gt(0)").hide(); //隐藏除了第一个以外的其他图片
    $("#next").click(function() //添加点击事件，下一页
    {
        $("#p_contant > li:eq("+page+")").fadeToggle(); //隐藏当前图片
        $("#p_O > span:eq("+page+")").text(".  "); //小圆点改为小点
        page ++; //目标转换成下个图片
        if (page >= 4) page=0; //一共4个图片，超出归零
        $("#p_contant li:eq("+page+")").fadeToggle(); //显示隐藏的后面一个图片
        $("#p_O > span:eq("+page+")").text("。"); //小点改为小圆点
    })
    $("#pre").click(function() //添加点击事件，上一页
    {
        $("#p_contant > li:eq("+page+")").fadeToggle(); 
        $("#p_O > span:eq("+page+")").text(".  ");
        page --; //目标转换成上个图片
        if (page <= 0) page=3;
        $("#p_contant li:eq("+page+")").fadeToggle();
        $("#p_O > span:eq("+page+")").text("。");
    })
})
```
