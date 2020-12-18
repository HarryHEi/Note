---
title: subprocess
date: 2018-1-23 17:15:37
tags: [python]
---

[文档](https://docs.python.org/3/library/subprocess.html)

该模块的底层流创建和管理由Popen类管理。

`class subprocess.Popen(args, bufsize=-1, executable=None, stdin=None, stdout=None, stderr=None, preexec_fn=None, close_fds=True, shell=False, cwd=None, env=None, universal_newlines=False, startupinfo=None, creationflags=0, restore_signals=True, start_new_session=False, pass_fds=(), *, encoding=None, errors=None)`

在新进程中执行子程序。
在POSIX，该类使用类似`os.execvp`的方式执行子程序。
在Windows，该类使用windows的`CreateProcess()`函数。

`args`参数需要是一个程序参数序列或者单个字符串。
如果`args`是个序列，默认将要执行的程序是`args`的第一个元素。
如果`args`是个字符串，字符串将作为名称或路径解析（没有附带参数），

Tips: `shlex.split()`可用于分解命令行
```
>>> import shlex, subprocess
>>> command_line = input()
/bin/vikings -input eggs.txt -output "spam spam.txt" -cmd "echo '$MONEY'"
>>> args = shlex.split(command_line)
>>> print(args)
['/bin/vikings', '-input', 'eggs.txt', '-output', 'spam spam.txt', '-cmd', "echo '$MONEY'"]
>>> p = subprocess.Popen(args) # Success!
```

`stdin`、`stdout`和`stderr`可以指定为以下值：

+ `PIPE`：创建到子进程的管道，通过`proc.stdin`、`proc.stdout`和`proc.stderr`进行流操作。

  ```
  >>> proc=subprocess.Popen(["python3"], stdin=subprocess.PIPE, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
  >>> proc.stdin.write(b'print("123")')
  12
  >>> proc.stdin.close()  # 必要的
  >>> proc.stdout.read()
  b'123\n'
  >>> proc.kill()
  >>> proc.wait()  # 必要的，否则proc会成为僵尸进程
  0
  ```

+ `DEVNULL`：相当于写入到`os.devnull`，`/dev/null` for POSIX, `nul` for Windows。
+ `None`：默认，集成父进程的文件标识。

`stderr`可以配置成`STDOUT`，与`stdout`共享文件句柄。

```
>>> proc=subprocess.Popen(["ls", "/opt/asd"], stdout=subprocess.PIPE, stderr=subprocess.STDOUT)
>>> proc.stdout.read()
b"ls: cannot access '/opt/asd': No such file or directory\n"
>>> proc.stderr == None
True
```

使用`os.kill`通过`PID`杀掉`proc`

```
>>> import os
>>> import signal
>>> os.kill(proc.pid, signal.SIGKILL)
>>> proc.wait()
```

`shell=True`指定使用`/bin/sh`运行命令，相当于：

```
Popen(['/bin/sh', '-c', args[0], args[1], ...])
```

