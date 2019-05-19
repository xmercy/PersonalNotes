## C语言一步编译

命令：`gcc -o hello.exe hello.c`

## C语言分步编译

1. 预处理

   > 命令：`gcc -E hello.c -o hello.i`

   - 宏定义展开
   - 头文件展开
   - 条件编译
   - 剔除注释

2. 编译

   > 命令：`gcc -S hello.i -o hello.s`

   - 检查语法
   - 将C语言转换为汇编语言

3. 汇编

   > 命令：`gcc -c hello.s -o hello.o`

   - 将汇编语言转换为机器语言

4. 链接

   > 命令：`gcc hello.o -o hello.exe`

   - 将C语言依赖库链接到程序中