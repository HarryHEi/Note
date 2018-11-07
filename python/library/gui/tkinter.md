---
title: Tkinter
date: 2016-03-27 14:58:50
tags: [python]
---

# 简介
在 GUI 程序中,会有一个顶层根窗口对象,它包含着所有小窗口对象,它们共同组成一个完整的 GUI 程序。
这些小窗口对象可以是文字标签、按钮、列表框等等。
这些独立的 GUI 构件就是所谓的组件。
所以当我们说创建一个顶层窗口的时候,我们实际上是指需要一个放置所有组件的地方。
# 示例
典型的 Python 语句如下行:
```
top = Tkinter.Tk()
```
进入无限循环等待用户操作:
```
Tkinter.mainloop()
```
# 组件

| 组件 | 描述 |
|-----|-------|
| Button | 按钮。类似标签,但提供额外的功能,例如鼠标掠过、按下、释放以及键盘操作/事件 |
| Canvas | 画布。提供绘图功能(直线、椭圆、多边形、矩形);可以包含图形或位图 |
| Checkbutton | 选择按钮。一组方框,可以选择其中的任意个(类似 HTML 中的 checkbox) |
| Entry | 文本框。单行文字域,用来收集键盘输入(类似 HTML 中的 text) |
| Frame | 框架。包含其他组件的纯容器 |
| Label | 标签。用来显示文字或图片 |
|Listbox | 列表框。一个选项列表,用户可以从中选择 |
| Menu | 菜单。点下菜单按钮后弹出的一个选项列表,用户可以从中选择 |
| Menubutton | 菜单按钮。用来包含菜单的组件(有下拉式、层叠式等等) |
|  Message | 消息框。类似于标签,但可以显示多行文本 |
| Radiobutton | 单选按钮。一组按钮,其中只有一个可被“按下”(类似 HTML 中的 radio) |
| Scale | 进度条。线性“滑块”组件,可设定起始值和结束值,会显示当前位置的精确值 |
| Scrollbar | 滚动条。对其支持的组件(文本域、画布、列表框、文本框)提供滚动功能 |
| Text | 文本域。多行文字区域,可用来收集(或显示)用户输入的文字(类似 HTML 中的 textarea) |
| Toplevel | 顶级。类似框架,但提供一个独立的窗口容器 |

```
#!/usr/bin/env python
# -*-coding:utf-8 -*-

from Tkinter import *

root = Tk()
label = Label(root, text='Hello World!')
label.pack()
mainloop()
```
```
quit = Button(root, text="Quit ", command=root.quit,bg='red', fg='white')
#bg:背景色红色，fg:前景色白色
```
对 packer 没有其他指令时,组件是按垂直顺序放置的(依次放在其他组件的上面)。
要水平放置则需要创建一个框架对象,再用它来添加按钮。

滑动
```
#!/usr/bin/env python
# -*-coding:utf-8 -*-

from Tkinter import *

def resize(ev=None):
    label.config(text='%d' % scale.get())

top = Tk()
top.geometry('250x150') #限定顶层窗口尺寸
label = Label(top, text='%d'%0)
label.pack(fill=Y, expand=1)
scale = Scale(top, from_=10, to=40, orient=HORIZONTAL, command=resize)
scale.set(12)
scale.pack(fill=X, expand=1)
quit = Button(top, text="QUIT",command=top.quit, activeforeground='white', activebackground='red')
quit.pack()
mainloop()
```

触发函数
```
#!/usr/bin/env python
#-*-coding:utf-8-*-

import Tkinter

top=Tkinter.Tk()
top.title("XZIT")
top.geometry('600x600')

def printname():
    mes.insert('1.0', "XZIT\n")

Tkinter.Button(top, text="开始", command = printname).pack()

mes = Tkinter.Text()
mes.pack()

Tkinter.mainloop()
```
