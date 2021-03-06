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

- `until/u 行号`

  执行到指定行 

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

- `thread apply all bt`

  查看所有线程的调用栈信息。此方法在多线程程序卡住时非常有用，快速找到调用栈异常的线程进行查看。

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

- `watch EXPRESSION`

  `watch *(int *)0x01`监视首地址在`0x01`处的4个字节中的数据何时被修改

- `x/FMT ADDRESS`

  根据格式将打印指定大小的指定数量的对象。 如果指定了负数，则内存为从地址向后检查。

  - ADDRESS是要检查的内存地址的表达式。
  - FMT是重复计数，后跟格式字母和大小字母。
  - 格式字母是o（八进制），x（十六进制），d（十进制），u（无符号十进制），t（二进制），f（浮点），a（地址），i（指令），c（字符），s（字符串）和z（十六进制，左边填零）。
  - 大小字母是b（字节），h（半字），w（字），g（巨大，8字节）。
  - 格式和大小字母的默认值是以前使用的默认值。默认计数为1。默认地址是最后打印的内容

  常用：`x/16cb`、`x/64dw`

- `set print pretty on/off`

  开启/关闭每行只会显示结构体的一名成员，并且根据成员的定义层次进行缩进

- `set scheduler-locking on/off`

  调试一个线程时，让其它线程暂停执行，off为关闭此功能

- `set print elements 0`

  设置gdb打印数组或字符串中元素的个数，0为打印整个数组或字符串。`show print elements `用于查看设置的值。

  

## 调试需要输入参数的程序

* `gdb --args 程序 参数1 参数2 ...`
* `r/run 参数1 参数2 ... `
* `set args 参数1 参数2 ...`

## 进行汇编指令级调试

1. `disassemble`显示源码对应的汇编指令

   `disassemble`显示当前所在栈帧源码对应汇编指令，`disassemble 函数名`显示指定函数源码对应的汇编指令，不包含源码，`disassemble/s 函数名`显示指定函数源码对应的汇编指令，包含源码

2. `set disassemble-next-line on/off`设置是否显示当前执行的行所对应的汇编指令。` show disassemble-next-line`显示disassemble-next-line的设置

3. `si/ni`对应`s/n`
4. `b *函数名`在函数对应的汇编指令第一行打断点，`b *汇编指令地址`在指定的汇编指令处打断点
5. `p $寄存器`查看寄存器中的值
6. `info reg`查看所有寄存器中的值
7. `x/i $pc`显示将要执行的汇编指令，`pc`计数器即程序计数器，用于存放下一条指令所在地址的地方

## 图形化界面使用

1. `ctrl + x, a`进入/退出图形化界面
2. `ctrl + l`刷新图形化界面，一般在图形化界面花屏时使用