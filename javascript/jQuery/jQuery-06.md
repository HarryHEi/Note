---
title: jQuery-06 Ajax
date: 2016-05-07 11:04:19
tags: [jquery]
---

# 简易服务器
使用python可以开启一个简易的服务器
```
python -m SimpleHTTPServer 8080
```

# load() 方法
载入远程HTML文件，并加载到DOM中，默认使用GET方式，如果传递参数，自动转为POST

+ 第一个参数是URL地址
+ 第二个参数是发送到服务器的数据
+ 第三个参数是回调函数(无论是否载入成功都会执行)

载入一个名为test.html的文件，加载到名为restest的元素
```
$("#restest").load("test.html")
```
可以筛选加载指定内容
```
$("#restest").load("index.html .dongmanRange a")
```
# .get()
通过GET请求载入信息
```
$(function()
{
    $("#test_send").click(function()
    {
        $.get("temp.html",function(data) //data就是返回的内容
        {
            alert(data);
        });
    })
})
```
# 结束
后面Ajax的内容涉及到PHP的JSON，等级太低，看不太懂，这本书的阅读就先到这边了~
总的来说，这本书是一本不错的工具书，不过使用的jQuery版本比较低，配合官方手册一起看更佳~
