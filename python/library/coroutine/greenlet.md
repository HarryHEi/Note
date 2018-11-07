---
title: greenlet
date: 2018-03-03 10:33:04
tags: [python]
---


greenlet是stackless的副产品，stackless是CPython的一个支持tasklets微线程的版本。

greenlet用于协程。与generators相比较的好处是，可以调用嵌套函数，以及不用使用yield关键字生成值。

# 示例

假如有个循环读取用户输入并打印的线程
```
def process_command():
    while True:
        value = read_next_char()
        print(value)

def read_next_char():
    value = input()
    return value
```

现在希望将这个功能插入到一个现有项目中，通过事件的方式调用`read_next_char()`，可能会考虑用独立的线程处理`process_command()`，但是用greelet是一种替代方案，没有相关锁定和关闭的问题

```
def process_command():
    while True:
        value = read_next_char()
        print(value)


def event_call(value):
    g_processor.switch(value)


def read_next_char():
    g_self = greenlet.getcurrent()
    value = g_self.parent.switch()
    return value


if __name__ == '__main__':
    g_processor = greenlet.greenlet(process_command)
    g_processor.switch()

    while True:
        line = input()
        event_call(line)
```

首先利用greelet启动`process_command()`函数，当调用`switch()`时开始执行，然后`process_command()` 调用`read_next_char()`，当执行到`parent.switch()`时g_processor挂起并切换到父greelet，也就是主线程，main下的while True就是一个主线程，当主线程调用`event_call(line)`时，该函数调用`switch()`切换到g_processor，继续执行挂起的`read_next_char()`并传递参数，`process_command`获得返回的值并打印。

# 用法
## 介绍

greenlet是一个独立的伪线程，可以想象成一个小的frame堆，底部frame是调用的初始函数(greenlet指定的函数)，顶部frame是当前暂停的函数。通过创建大量这样的堆并且在它们之间切换以完成任务，注意不会隐式切换，需要明确调用。

当创建一个greenlet时，获得了一个空堆，当switch到它时，开始运行这个greenlet指定的函数，这个函数可能调用其他函数或者切换到其他greenlet。当堆底部函数结束调用时，这个greenlet的堆又变为空，然后就认为这个greenlet结束生命周期，greenlet也可能因为未捕获异常结束生命周期。

例如：
```
def test1():
    print(1)
    gr2.switch()
    print(2)


def test2():
    print(3)
    gr1.switch()
    print(4)


gr1 = greenlet.greenlet(test1)
gr2 = greenlet.greenlet(test2)
gr1.switch()
```

首先创建了两个greenlet，当调用`gr1.switch()`时，执行指定的`test1()`函数，打印一个1，然后挂起切换到`test2()`，打印一个3，然后切换回`test1()`，继续之前挂起的内容，打印2，`test1()` 执行完成，gr1现在完成了生命周期，回到`gr1.switch()`语句，然后程序运行结束，这里4不会被打印出来。

## Parents

每个greenlet有一个父greenlet，父greenlet在创建这个greenlet的地方(可以在任何时候修改)初始化，当greenlet完成完成生命周期后会回到父greenlet执行。这些greenlet通过这种方式组织为树结构，而不在用户创建的greenlet中运行的顶层代码为树的根，隐藏在main greenlet中。

上面的例子中，gr1和gr2都把main greenlet作为父项，每当其中一个生命周期结束，执行回到main。

没有捕获的异常也会传播到父项。比如如果上例中test2抛出异常，会把gr2杀掉并传递异常给父项，也就是main greenlet，虽然是由test1切换给test2，但traceback将显示test2异常，而不是test1。

## 实例

`greenlet.greenlet(run=None, parent=None)`
greenlet object，run参数指定调用对象，parent指定父greenlet，默认指定当前greenlet。

`greenlet.getcurrent()`
获取当前greenlet。

`greenlet.GreenletExit`
这个特殊的异常不会传递给父项，用于杀死某个greenlet。

greenlet可以被继承，一个greenlet通过调用run属性运行，子类也可以定义一个名为run的方法替代run属性。

## Switching
 
多个greenlet之间，当一个greenlet的switch()被调用时进行切换，这种情况下，执行跳转到调用switch()的greenlet，或者当一个greenlet完成生命周期，这种情况下，执行跳转到父greenlet。切换时，对象或者异常被发送给目标greenlet，可用于在greenlet之间传递消息。

例如

```
def test1(x, y):
    z = gr2.switch(x + y)
    print(z)


def test2(z):
    z *= 2
    gr1.switch(z)


gr1 = greenlet.greenlet(test1)
gr2 = greenlet.greenlet(test2)
gr1.switch(1, 2)
```

这里打印出6。

`g.switch()`
切换执行并发送参数给greenlet `g`，特殊情况下，如果`g`没有在执行，这个调用会使其开始运行。

Dying greenlet
如果一个greenlet的run()调用结束，它的返回值会被发送给父项，如果run()由于异常中断，异常会被传递给父项(除非是greenlet.GreenletExit)。

除了以上几种情况，目标greenlet通常接收那个挂起的对象的传入值作为switch()的返回值，事实上，尽管switch调用不会立即返回，但它会在未来的某个时间点切换回来的时候返回。当出现这种情况，暂停将在switch()之后回复，并且switch()本身会返回刚刚发送的对象。意味着，`x = g.switch(y)`会发送`y`给`g`，并且随后会将一些其他greenlet传递的对象返回赋值给`x`。

## greenlet的方法和属性

`g.switch(*args, **kwargs)`
挂起当前greenlet，切换执行到greenlet `g`。

`g.run`
`g`的可调用对象，将在开始时调用。

`g.parent`
父parent，是可写的，但是不允许创建环状父项。

`g.gr_frame`
当前顶部frame，或者None，比如上例中，当从test1切换到test2时，gr1的顶部frame就是刚刚暂停的test1。而test2正在执行，没有顶层frame，所以`gr2.gr_frame`为None。

`g.dead`
判断`g`有没有完成生命周期

`bool(g)`
如果g是活跃的，返回True，如果已经完成生命周期或者还没开始，返回False。

`g.throw([typ[, val[, tb]]])`
挂起当前greenlet，切换执行到`g`，但是在`g`中立刻抛出给与的异常。如果没有提供参数，默认为greenlet.GreenletExit。效果等同于:

```
def raiser():
    raise typ, val, tb
g_raiser = greenlet(raiser, parent=g)
g_raiser.switch()
```

## Greenlets和Python threads

greenlets可以和Python threads结合使用，这种情况下，每个线程有个独立的main greenlet以及独立的树结构。不同线程的greenlet之前不能混合或切换。

## 垃圾回收greenlets

如果一个greeblet的所有引用都消失(包括父项的以及其他greenlet的引用)，那么就没办法切换回这个greenlet。这种情况下，greenlet生成一个GreenletExit异常，这是greenlet收到异步执行的唯一情况。这给与`try:finally:`有机会清理greenlet持有的资源。这个特性支持使用无限循环等待数据并处理的编程风格，这个循环会在所有引用消失时中断。

一个greenlet期望于要么结束生命周期要么由于一个新的引用复苏，仅仅捕获并或略GreenletExot异常往往导致一个无限循环。

greenlet不参与垃圾回收，greenlet内部的循环引用将不会被检测到，保存greenlet的循环引用将导致内存泄漏。

## 跟踪支持

标准Python跟踪和分析对greenlet不起作用，因为帧和堆在同一Python线程。使用传统方法很难检测到greenlet切换，所以为了改善调试，greenlet模块有一些新的函数用于追踪和分析greenlet代码。

`greenlet.gettrace()`
返回一个之前设置的跟踪函数或者None。

`greenlet.settrace(callback)`
设置一个新的跟踪函数，并且返回一个之前设置的跟踪函数，或者None。callback在各种事件中被调用，预计有一下签名:
```
def callback(event, args):
    if event == 'switch':
        origin, target = args
        # 处理一个从origin到target的switch调用。
        # 注意callback在target greenlet上下文运行。
        # 异常使用target.throw()传递。
    if event == 'throw':
        origin, target = args
        # 处理一个从origin到target的throw调用。
        # Note that callback is running in the context of target
        # greenlet and any exceptions will replace the original, as
        # if target.throw() was used with the replacing exception.
        return
```

为了兼容性，只有当event是switch或者throw时对args解包才是重要的，这个API可以被扩展到新的类似`sys.settracfe()`的事件。
