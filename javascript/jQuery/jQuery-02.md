---
title: jQuery-02 jQuery选择器
date: 2016-05-01 15:37:48
tags: [jquery]
---

# jQuery选择器
选择器是jQuery的根基，在jQuery中，对事件处理，遍历DOM和Ajax操作都依赖于选择器

jQuery中的选择器完全继承了CSS风格

使用jQuery选择器不仅比传统DOM操作简介，并且能避免错误

## 基本选择器

选择器 | 描述 | 返回 | 示例
-----------|-------|--------|-------
#id | 根据给定id匹配一个元素 | 单个元素 | $("#test")
.class | 根据类匹配元素 | 集合元素 | $(".test")
element | 根据给定元素名匹配元素 | 集合元素 | $("p")
* | 匹配所有元素 | 匹配所有元素 | 集合元素 | $("*")
a,b,c,... | 匹配每个元素 | 集合元素 | $("div,p,.myclass")

可以使用基本选择器完成大部分工作

## 层次选择器

示例 | 描述
------|------
$("div span") | 选取div里面所有span子元素
$("div > span") | 选组div里面span子元素
$(".one + div") | class为one的下一个兄弟元素
$(".one ~ div") | class为one的后面所有兄弟元素

第一个和第二个比较常用，后面两种方法可以用更加简单的代替
使用`next()`方法代替
```
$(".one").next("div");
```
使用`nextAll()`方法代替
```
$(".one").nextAll("div")
```
`siblings()`与前后位置无关，只要是同辈节点都能匹配
```
$(".one").sibling("div")
```

## 过滤选择器
### 基本过滤选择器

示例 | 描述
-------|-------
$("div:first") | div元素中的第一个div元素
$("div:last") | div元素的最后一个div元素
$("div:not(.myClass)") | 选取class不是myclass的div元素
$("div:even") | 选取索引是偶数的div元素(0开始)
$("div:odd") | 奇数
$("div:eq(index)") | 选取索引等于index的div元素
$("div:gt(index)") | 大于
$("div:lt(index)") | 小于
$("div:header") | 选取所有h1,h2,h3...
$("div:animated") | 选取当前正在执行动画的所有元素

### 内容过滤选择器
内容过滤选择器主要体现在它所包含的子元素或文本内容上

示例 | 描述
--------|-------
$("div:contains('你')") | 选取内容含有‘你’的元素
$("div:empty") | 选取不包含子元素的div空元素
$("div:has(p)") | 选取含有p元素的div元素
$("div:parent") | 选取拥有子元素(包括文本元素)的div元素

### 可见性过滤选择器
可见性过滤器是根据元素的可见和不可见状态来选择相应的元素

示例 | 描述
--------|-------
$(":hidden") | 选取所有不可见元素
$("div:visible") | 选取所有可见div元素

显示所有隐藏的元素，show()和hide()可以有两个参数，分别是速度(ms)和结束后执行的函数
```
$(document).ready(function()
{
    $v = $(":hidden");
    $v.show();
})
```
### 属性过滤选择器
属性过滤器的过滤规则是通过元素的属性来获取相应的元素

示例 | 描述
--------|-------
$("div[id]") | 选取具有属性id的div元素
$("div[title=test]") | 选取title为"test"的div元素
$("div[title!=test]") | 不为
$("div[title^=t]") | 以"t"开始
$("div[title$=t]") | 以"t"结束
$("div[title*=e]") | 含有"t"
$("div[...][...]") | 允许多个过滤器复合

### 子元素过滤选择器

选择器 | 描述
--------|-------
:nth-child(index/even/...) | 每个父元素下第index个子元素(1开始)
:first-child | 每个父目录下第一个子元素
:last-child | 最后一个
:only-child | 是唯一子元素的子元素

使用css()方法改变class为'haschildren'的第一个子元素的字体大小
```
$(document).ready(function()
{
    var $v = $(".has_children:first-child");
    $v.css("font-size","30px");
})
```

### 表单对象属性过滤选择器
此选择器主要对所选择的表单元素进行过滤

选择器 | 描述
--------|-------
:enabled | 选取所有可用元素
:disabled | 选取所有不可用元素
:checked | 选取所有被选中元素(单选框，复选框)
:selected | 选取所有被选中元素(下拉表)

## 表单选择器
jQuery加入了表单选择器，能方便地获得表单的某个或某类型元素

选择器 | 描述
--------|-------
:input | 选取所有input、textarea、select和button元素
:text | 选取所有单行文本框
:password | 密码框
:radio | 单选框
:checkbox | 多选框
:submit | 提交按钮
:image | 图像按钮
:reset | 重置按钮
:button | 所有按钮
:file | 上传域
:hidden | 不可见元素

# 选择器中的注意事项
## 特殊字符
选择器中如果含有'.'、'#'、'('或者']'等，需要使用转义字符转义

找到href为‘#’的元素
```
$(document).ready(function()
{
    var $v = $("[href=\\#]"); //转义字符'#'
    alert($v.length)
})
```
## 选择器中含有空格
缺少或者添加空格，含义可能因此而变得不一样
```
$(".links:hidden") //隐藏的并且class为‘links’的元素
$(".links :hidden") //class为‘links’的元素里面的隐藏元素(子元素)
```
# 实例
建立一个显示动漫排行的文档
```
<div class="dongmanRange">
    <h2>动漫排行</h2>
    <ul>
        <li>1 <a href="#">海贼王</a></li>
        <li>2 <a href="#">七龙珠</a></li>
        <li>3 <a href="#">火影忍者</a></li>
        <li>4 <a href="#">死神</a></li>
        <li>5 <a href="#">名侦探柯南</a></li>
        <li>6 <a href="#">妖精的尾巴</a></li>
        <li>7 <a href="#">东京食尸鬼</a></li>
        <li>8 <a href="#">银魂</a></li>
        <li>9 <a href="#">刀剑神域</a></li>
        <li>10 <a href="#">约会大作战</a></li>
    </ul>
    <div class="showmore">
        <a href="#">显示全部</a>
    </div>
</div>
```
实现以下效果

+ 一开始只显示前五个，后面的全部隐藏
+ 当用户点击显示全部按钮之后，显示全部列表，高亮显示推荐，并把按钮内容改为“收回”
+ 当用户点击“收回时”，隐藏6个以外的所有列表，并把按钮内容改为“显示全部”

JavaScript代码如下
```
$(document).ready(function() //等待DOM加载完成
{
    var $overtop5 = $("ul li:gt(4)"); //选取ul下所有li元素，筛选出li元素index大于4的
    $overtop5.hide(); //隐藏

    var $button = $(".showmore > a"); //选取class为showmore的子元素a
    $button.click(function() //添加onclick事件
    {
        if ($overtop5.is(":hidden")) //如果属性是隐藏
        {
            $overtop5.show(); //显示
            $button.text("收回"); //按钮内容改为"收回"
            $("ul li").filter(":contains('火影'),:contains('刀剑'),:contains('银魂')") //filter参数可以是多个选择器的集合
                .addClass("promoted"); //高亮
            return false; //阻止链接跳转
        }
        else
        {
            $overtop5.hide(); //隐藏
            $button.text("全部显示"); //按钮内容改为"全部显示"
            $("ul li").filter(":contains('火影'),:contains('刀剑'),:contains('银魂')") //filter参数可以是多个选择器的集合
                .removeClass("promoted"); //去除高亮
            return false; //阻止链接跳转
        }
    })
})
```
