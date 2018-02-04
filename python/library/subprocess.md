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
