# GDB使用

## 常用指令

- `list/l`

  列出源码，可指定行号

- `break/b 行号`

  在指定行打断点

- `info b`

  查看断点信息

- `next/n`

  逐过程

- `step/s`

  逐语句

- `run/r`

  运行程序

- `quit/q`

  退出gdb调试

- `continue`

  执行到下一断点

- `print/p 变量名`

  打印变量值

- `finish`

  跳出函数内部，返回调用处

- `start`

  从程序入口函数处开始执行，并且停在程序入口第一行代码。不能使用`s`，`s`是`step`指令的简写。

- `ptype 变量名`

  查看变量类型

- `info thread`

  查看所有线程信息

- `thread 线程号`

  切换到指定线程

- `backtrace/bt`

  查看当前线程的调用栈信息

- `frame 栈帧号` 

  切换到指定栈帧编号的栈帧上下文，查看指定栈帧上下文中的变量

- `up/down`

  向上/下切换栈帧

- `display 变量名`

  监视指定变量

- `undisplay 监视变量的编号`

  取消监视

- `set follow-fork-mode child/parent`

  设置跟踪父进程还是子进程，必须在fork函数调用前设置。

## 调试需要输入参数的程序

* `gdb --args 程序 参数1 参数2 ...`
* `r/run 参数1 参数2 ... `
* `set args 参数1 参数2 ...`