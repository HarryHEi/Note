---
title: javascript-09 动画
date: 2016-04-30 16:14:36
tags: [javascript]
---

# JavaScript实现动画效果
动画就是让元素的位置随着时间不断发生变化

## 位置
网页元素在浏览器窗口里的位置是一种表示性的信息，通常由CSS负责设置
```
example
{
    position: absolute;
    top: 50px;
    left: 100px;
}
```
DOM代码如下
```
example.style.position = "absolute";
example.style.left = "100px";
example.style.top = "50px";
```
posion合法值有static、fixed、relative、和absolute四种

static是默认值，有关元素会按照在标记的先后顺序出现在窗口

relative与static相似，区别是还可以通过float属性从文档的正常显示脱离开

如果某个元素position属性设置为absolute，就可以放到容器的任何位置，容器要么是文档本身，要么是有一个fixed或者absolute属性的父元素，显示位置由top、left、right、bottom等元素决定，可以用百分比或者像素设置这些元素属性值

创建一个标签
```
<div id="cartoon">
    <p>hello harryx</p>
</div>
```
使用JavaScript指定位置
```
addLoadEvent(
    function positionCartoon()
    {
        var cartoon = document.getElementById("cartoon");
        cartoon.style.position = "absolute";
        cartoon.style.left = "50px";
        cartoon.style.top = "100px";
    }
)
```
改变元素的位置就是执行函数去改变这个元素的style.top或者style.left等属性
```
function moveCartoon()
{
    var cartoon = document.getElementById("cartoon");
    cartoon.style.left = "200px";
    cartoon.style.top = "50px";
}
```

## 事件
JavaScript函数setTimeout能够让某个函数经过一段预定时间后才开始执行
```
setTimeout("function", interval)
```
第一个参数是一个字符串，是将要执行的函数的名字
第二个参数是一个数值，以毫秒为单位设定了需要多长时间后才开始执行第一个参数给出的函数

把函数的调用赋值给一个变量，之后可以使用clearTimeout函数取消等待执行的某个函数
```
variable = setTimeout("function", interval)
clearTimeout(variable)
```

修改一下positionCartoon函数，让它隔一段时间后执行moveCartoon函数
```
addLoadEvent(
    function positionCartoon()
    {
        var cartoon = document.getElementById("cartoon");
        cartoon.style.position = "absolute";
        cartoon.style.left = "50px";
        cartoon.style.top = "100px";
        movement = setTimeout("moveCartoon()", 1000);
    }
)
```
让元素渐变移动，主要逻辑如下

1. 获得元素的当前位置
2. 如果元素已经到达目的地，退出这个函数
3. 如果尚未到达目的地，则把它向目的地移近一点
4. 经过一段时间后重复步骤1

注意，在获得当前位置时是以字符串形式返回，要使用parseInt()函数提取数值
```
function moveCartoon()
{
    var cartoon = document.getElementById("cartoon");
    var xpos = parseInt(cartoon.style.left);
    var ypos = parseInt(cartoon.style.top);
    if (xpos == 200 && ypos == 100) return true;
    if (xpos < 200) xpos++;
    if (xpos > 200) xpos--;
    if (ypos < 100) ypos++;
    if (ypos > 100) ypos--;
    cartoon.style.left = xpos + "px";
    cartoon.style.top = ypos + "px";
    movement = setTimeout("moveCartoon()", 50);
}
```

## 抽象
### 创建moveElement函数
与之前的函数不同的是，moveElement函数可以输入参数

+ 打算移动的元素id
+ 该元素的目的地
+ 两次移动的间隔时间

定义一个moveElement函数
```
function moveElemeng(elementID, finalX, finalY, interval)
{
    var elem = document.getElementById(elementID);
    var xpos = parseInt(elem.style.left);
    var ypos = parseInt(elem.style.top);
    if (xpos == finalX && ypos == finalY) return true;
    if (xpos < finalX) xpos++;
    if (xpos > finalX) xpos--;
    if (ypos < finalY) ypos++;
    if (ypos > finalY) ypos--;
    elem.style.left = xpos + "px";
    elem.style.top = ypos + "px";
    repeat = "moveElemeng("+"'"+elementID+"'"+","+finalX+","+finalY+","+interval+")";
    moment = setTimeout(repeat, interval)    
}
```

### 调用
```
addLoadEvent(
    function positionCartoon()
    {
        var cartoon = document.getElementById("cartoon");
        cartoon.style.position = "absolute";
        cartoon.style.left = "50px";
        cartoon.style.top = "100px";
        moveElemeng("cartoon",200,100,10)
    }
)
```

### 完整代码
```
$(document).ready(function()
    {
        var logop = document.getElementById("logop");
        logop.style.left = "200px";
        logop.style.top = "0px";
        moveElemeng("logop",200,30,5);
    }
)

function moveElemeng(elementID, finalX, finalY, interval)
{
    var elem = document.getElementById(elementID);
    var xpos = parseInt(elem.style.left);
    var ypos = parseInt(elem.style.top);
    if (xpos == finalX && ypos == finalY) return true;
    if (xpos < finalX) xpos++;
    if (xpos > finalX) xpos--;
    if (ypos < finalY) ypos++;
    if (ypos > finalY) ypos--;
    elem.style.left = xpos + "px";
    elem.style.top = ypos + "px";
    repeat = "moveElemeng("+"'"+elementID+"'"+","+finalX+","+finalY+","+interval+")";
    moment = setTimeout(repeat, interval)    
}
```

# HTML5
HTML5添加了新的标记元素

## Canvas
通过canvas元素标签可以实现与静态图片的交互，创建一个画布
```
<canvas id="draw-in-me" width="120" height="40">
</canvas>
```
然后使用JavaScript作画
```
addLoadEvent( 
    function draw()
    {
        var canvas = document.getElementById("draw-in-me");
        if (canvas.getContext)
        {
            var cxt=canvas.getContext("2d");
            cxt.moveTo(10,10);
            cxt.lineTo(150,50);
            cxt.lineTo(10,50);
            cxt.stroke();
        }
    }
)
```

## 音频和视频
HTML5的video元素为在文档中嵌入影片以及与影片的交互提供了一种标准方式，同时也把操作简化成一个标签

video 元素为在文档中嵌入影片以及与影片交互定义了一种标准方式，同时也简化成一个标签
这里嵌入了一个mp4视频，同时给出了浏览器不支持video时的下载链接
```
<video src="./static/mv/badapple.mp4">
    <!-- 不支持时替换内容 -->
    <a href="./static/mv/badapple.mp4">Download movie.mp4</a>
</video>
```
audio元素用法也差不多
```
<audio src="./static/mv/badapple.mp3">
    <!-- 不支持时替换内容 -->
    <a href="./static/mv/badapple.mp3"></a>
</audio>
```
为保证每个人都能看到视频，必须作多种格式的视频并在video元素中包含多个来源

为了确保HTML5的兼容性，至少包含下列三个版本

+ 基于H.264和ACC的MP4
+ WebM (VP8 + Vorbis)
+ 基于Theora视频和Vorbis音频的Ogg文件

# 总结
这本书就读到这边了，总的来说是本通俗易懂的书，对js的DOM编程有了一定的概念，也稍微了解了下HTML5，那么，继续学习吧！
