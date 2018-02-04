---
title: jQuery-05 表单和表格
date: 2016-05-04 14:33:25
tags: [jquery]
---

# 表单应用
之前通过focus和blur方法设置表单默认值时就是一个例子，下面还有其他例子
## 多行文本框
首先创建一个表单
```
<form>
    <div class="msg">
        <div class="msg_con">
            <span class="bigger">放大</span>
            <span class="smaller">缩小</span>
        </div>
        <div>
            <textarea id="editarea" rows="8" cols="20">--</textarea>
        </div>
    </div>
</form>
```
要实现的功能如下

+ 当点击放大按钮之后，如果高度小于500px就增大50px
+ 当点击缩小按钮之后，如果高度大于50px就减少50px

```
$(function()
{
    var $editarea = $("#editarea"); //找出表单输入框
    $(".bigger").click(function() //放大
    {
        if ($editarea.height() < 500)
        {
            $editarea.height($editarea.height() + 50); //高度加50
        }
        return false;
    });
    $(".smaller").click(function() //缩小
    {
        if ($editarea.height() > 50)
        {
            $editarea.height($editarea.height() - 50); //高度减50
        }
        return false;
    })
})
```
点击按钮之后，高度迅速变化
为了美观，也可以使用动画
```
$(function()
{
    var $editarea = $("#editarea");
    $(".bigger").click(function()
    {
        if ($editarea.height() < 500)
        {
            $editarea.animate({"height":"+=50"},"slow");
        }
        return false;
    });
    $(".smaller").click(function()
    {
        if ($editarea.height() > 50)
        {
            $editarea.animate({"height":"-=50"},"slow");
        }
        return false;
    })
})
```
操控滚动条是使用的scrollTop属性
通过按钮实现，滚动条上下移动
```
$editarea.animate({"scrollTop":"+=50"},"slow");
```
## 复选框
最基本的应用是对复选框的全选、反选操作
主要通过复选框选诉的checked属性来设置
当checked属性为true时，说明被选中，反之没有

首先建立一个表单，有一些复选框和一个全选复选框
当勾选"全选"复选框时，全部勾选
取消勾选"全部"复选框时，全部不勾选
按下提交按钮时，获取到checked属性为true的复选框的值，然后都打印出来
```
<div>
    <form>
        you like ? <br / >
        <input type="checkbox" id="asd" name="items" value="萝莉" / > 萝莉 <br / >
        <input type="checkbox" name="items" value="御姐" / > 御姐 <br / >
        <input type="checkbox" name="items" value="三无" / > 三无 <br / >
        <input type="checkbox" name="items" value="变态" / > 变态 <br / >
        <input type="checkbox" name="items" value="病娇" / > 病娇 <br / >
        <input type="checkbox" id="selectall_box" value="全选" / > 全选  <br / >
        <input type="button" id="send_box" value="提交" / > <br / >
    </form>
</div>
```
这里用到了prop方法更改checked属性，因为jQuery的库的兼容问题，如果使用attr会出现第二次不能勾选的情况
```
$("...").prop("checked"); //用于判断
$("...").prop("disabled", false); //设置为不勾选
$("...").prop("checked", true); //设置为勾选
```
遍历每个对象使用了each()方法
```
$(function()
{
    $("#selectall_box").click(function()  //全选复选框
    {
        if ($(this).prop("checked")) //勾选
        {
            $("[name=items]:checkbox").prop("checked",true);
        }
        else //取消勾选
        {
            $("[name=items]:checkbox").prop("checked",false);
        }
    });
    $("#send_box").click(function() //提交
    {
        var likelis = "you like:\r\n";
        $("[name=items]:checkbox:checked").each(function() //使用each遍历
        {
            likelis += $(this).val() + "\r\n"; //val()获得表单值
        });
        alert(likelis);
    })
})
```
## 下拉框
主要实现下面的功能

+ 能将选中的选项添加给对方
+ 能将全部选项添加给对方
+ 双击添加

首先在文档中添加下拉框
一共有两个下拉框，"multiple"显示全部元素
```
<div class="select_path">
    <select multiple id="select_test1">
        <option value="1">1</option>
        <option value="2">2</option>
        <option value="3">3</option>
        <option value="4">4</option>
        <option value="5">5</option>
    </select>
    <select multiple id="select_test2">
    </select>
    <div>
        <a href="#" id="select_add">添加</a>
        <a href="#" id="select_all">全部添加</a>
        <a href="#" id="select_remove">删除</a>
        <a href="#" id="select_removeall">全部删除</a>
    </div>
</div>
```
JavaScript代码如下
```
$(function()
{
    $("#select_add").click(function() //添加
    {
        var $option = $("#select_test1 option:selected");
        $option.appendTo("#select_test2"); //移动追加
        return false;
    })
    $("#select_all").click(function() //全部添加
    {
        var $options = $("#select_test1 option");
        $options.appendTo("#select_test2");
        return false;
    })
    $("#select_remove").click(function() //移除
    {
        var $option = $("#select_test2 option:selected");
        $option.appendTo("#select_test1");
        return false;
    })
    $("#select_removeall").click(function() //全部移除
    {
        var $options = $("#select_test2 option");
        $options.appendTo("#select_test1");
        return false;
    })
    $("#select_test1").dblclick(function() //双击添加
    {
        var $option = $("#select_test1 option:selected");
        $option.appendTo("#select_test2");
    })
    $("#select_test2").dblclick(function() //双击删除
    {
        var $option = $("#select_test2 option:selected");
        $option.appendTo("#select_test1");
        return false;
    })
})
```
