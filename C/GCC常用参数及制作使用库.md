# GCC常用参数及制作使用库

## 常用参数

1. `-o`

   指定输出文件名

   `gcc hello.c -o hello`

2. `-I`

   指定头文件所在目录

   `gcc hello.c -o hello -I ./includes`

3. `-g`

   添加调试信息，配合gdb使用

   `gcc main.c -o debug_main -g -I ./includes/`

4. `-Wall`

   显示所有调试信息

   ` gcc main.c -o main -I ./includes/ -g -Wall`

5. `-D`

   向程序中动态注册宏定义

   `gcc main.c -o main -I ./includes/ -g -Wall -D HELLO`

## 制作与使用静态库、动态库

- 制作静态库

  1. 生成`.o`文件（c编译第三阶段-汇编）

     `gcc -c xmath.c -o xmath.o`

  2. 利用生成的`.o`文件制作静态库，库名的命名规则：`lib库名.a`

     `ar rcs libxmath.a xmath.o`

- 使用静态库

  1. 引入静态库及静态库对应的头文件到项目中

  2. 编译时将静态库与项目源文件一起编译，注意静态库一定要放在源文件参数后

     `gcc test.c ./lib/libxmath.a -o test -I ./include/ -g -Wall`

- 制作动态库

  1. 生成`.o`文件，与静态库不同的是**必须加上 `-fPIC`参数**

     `gcc -c xmath.c -o xmath.o -I ./includes/ -Wall -g -fPIC `

     > PIC：position independent code 与位置无关的代码

  2.  利用生成的`.o`文件制作动态库，库名的命名规则：`lib库名.so`

     `gcc xmath.o -o libxmath.so -shared`

- 使用动态库

  1. 引入头文件及动态库

  2. 编译

     `-l`指定库名，`-L`指定动态库所在目录。这个参数**必须放在项目源文件参数后面，否则报错**

     ` gcc main.c -o main -l xmath -L ./lib/ -I ./includes/ -g -Wall `

## 动态库加载出错

- 链接器

  工作于链接阶段，需要`-l`及`-L`参数支持

- 动态链接器

  工作于程序运行阶段，工作时需要提供动态库所在目录位置，不提供则执行会报错

  1. 方法一：` export LD_LIBRARY_PATH=./lib`

     此方法是临时的，当关闭终端后在打开终端，加载动态库的环境变量会初始化，动态库加载仍然会报错。

  2. 方法二：修改用户家目录下的`.bashrc`文件

     在最后一行添加` export LD_LIBRARY_PATH=./lib`保存退出，然后执行`source .bashrc`命令是配置文件生效

- 查看程序依赖的动态库加载路径

  `ldd 可执行程序名`