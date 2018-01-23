[文档](https://docs.python.org/3/library/os.html)

`os.wait()`
等待一个子进程完成，然后返回一个包括其pid和返回状态值的元组。


`os.system(command)`

在子Shell执行一个命令串，通过调用标准C函数`system()`实现。

如果调用的串中含有中文，需要先encode()成utf-8编码的bytes字符串，然后调用该方法。

在Unix，返回值是os.wait()指定格式。

`os.popen(cmd, mode='r', buffering=-1)`

从命令cmd打开一个管道，返回值是个连接到管道的文件对象，权限取决于mode参数，

`os.fsync(fd)`

强制将文件写入磁盘。

在Unix，这个会调用本地的fsync()函数，
在Windows，调用MS的_commit()函数

如果有一个文件对象f，先调用f.flush()，然后调用os.fsync(f.fileno())


`os.execl(path, arg0, arg1)`
`os.execle(path, arg0, arg1, ..., env)`
`os.execlp(file, arg0, arg1, ...)`
`os.execv(path, args)`
`os.execve(path, args, env)`
`os.execvp(file, args)`
`os.execvpe(file, args, env)`

这些函数运行一个新的程序代替当前进程，没有返回值。

在Unix，新的程序会加载到当前进程，并且与调用者具有相同进程id，错误会通过OSError报告。

进程会被立即替换，打开的文件对象和描述符不会刷新，所以如果如果缓存了数据，需要在调用`exec*`函数之前使用`sys.stdout.flush()`或`os.fsync()`刷新。

`l`和`v`结尾的`exec*`函数的区别是命令行参数的传递方式。
`l`方式各参数简单的成为`exec*`功能的附加参数。
`v`用于处理参数数量可变的情况。
