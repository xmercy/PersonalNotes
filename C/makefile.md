# makefile

> `makefile`文件名默认是`makefile`或`Makefile`，若不是，make命令需要带上`-f`参数指定makefile文件名

## 规则

makefile文件的内容是一组规则的集合，规则格式如下：

```makefile
目标文件:生成目标文件的依赖
	通过依赖生成目标文件的命令（必须有一个tab的缩进）
```

注意：makefile中的tab不能是对应数量的空格组合，否则报错

```makefile
ALL:hello

hello.o:hello.c
    gcc -c hello.c -o hello.o -I ./inc -Wall
xadd.o:xadd.c
    gcc -c xadd.c -o xadd.o -I ./inc -Wall
xsub.o:xsub.c
    gcc -c xsub.c -o xsub.o -I ./inc -Wall
xmul.o:xmul.c
    gcc -c xmul.c -o xmul.o -I ./inc -Wall
xdiv.o:xdiv.c
    gcc -c xdiv.c -o xdiv.o -I ./inc -Wall
hello:hello.o xadd.o xsub.o xmul.o xdiv.o
    gcc hello.o xadd.o xsub.o xmul.o xdiv.o -o hello -I ./inc -Wall
```

## 工作原理

1. 若要生成目标文件，先检查规则中生成该目标文件的依赖文件是否存在，若不存在，则查找是否有规则来生成该依赖文件。
2. 检查规则中的目标是否需要更新，必须检查其所有的依赖，任何一个被更新（其修改时间会比目标文件新），则目标文件会被重新生成。

> 第二条工作原理可以实现在多文件编译时，不需要重新编译所有文件，只需要编译有改动的文件，即可完成编译，提高编译效率。

## 函数

1. 通配符展开函数：`$(wildcard <pattern>)`

   功能：查找当前目录下所有符合模式`<pattern>`的文件，将文件名组成字符串，多个文件名之间以空格隔开。返回文件名组成的字符串。

   示例：`src = $(wildcard ./*.c)`，匹配当前目录下所有`.c`文件，并将文件名组成字符串返回。若当前目录下有`a.c`、`b.c`，则返回值为`a.c b.c`

2. 模式字符串替换函数：`$(patsubst <pattern>,<replacement>,<text>)`

   功能：查找`<text>`中的单词（单词以“空格”、“Tab”或“回车”“换行”分隔）是否符合模式`<pattern>`，如果匹配的话，则以`<replacement>`替换。返回被替换过后的字符串。

   注意：`<pattern>`可以包括通配符“%”，表示任意长度的字串。如果`<replacement>`中也包含“%”，那么，`<replacement>`中的这个“%”将是`<pattern>`中的那个“%”所代表的字串。

   示例：`obj = $(patsubst %.c, %.o, $(src))`，`src`中所有匹配`%.c`的词，全部替换为`%.o`，并返回。这里`$(src)`是取`src`变量的值。若`src`值是`a.c b.c`,则返回值为`a.o b.o`。

3. 去除文件目录函数：`$(notdir <text>)`

   功能：去除文件的目录部分，仅保留文件名

   示例：`$(notdir src/foo.c hacks)`返回值是“foo.c hacks”。

## 自动变量

> 自动化变量只应出现在规则的命令中。

1. `$@`表示规则中的目标

   注意：在模式规则中，此变量表示所有的目标的挨个值，遍历每一个值套用规则

2. `$<`表示规则中的第一个依赖项目

   注意：在模式规则中，此变量表示所有的依赖的挨个值，遍历每一个值套用规则

3. `$^`表示规则中所有依赖项目，多个依赖项目将以空格分隔，有重复依赖项会消除重复项

## 模式规则

1. 隐式模式规则

   make会试图去自动推导产生目标的规则和命令，如果make可以自动推导生成这个目标的规则和命令，那么这个行为就是隐含规则的自动推导。

   ```
   hello.o:hello.c
       gcc -c $< -o $@ -I ./inc -Wall
   #%.o:%.c
   #   gcc -c $< -o $@ -Wall
   hello:$(obj)
       gcc $^ -o $@ -Wall
   ```

   如上面的例子中，hello依赖的不仅仅是hello.o，但我们仅仅只定义呢hello.o的生成规则，那么其他的依赖项如何生成？这里就会用到make的隐式规则。最终make时，输出如下：

   ```
   mxc@xxxxxx:~/Projects/MakefileTest$ make
   cc    -c -o xmul.o xmul.c
   cc    -c -o xdiv.o xdiv.c
   gcc -c hello.c -o hello.o -I ./inc -Wall
   cc    -c -o xsub.o xsub.c
   cc    -c -o xadd.o xadd.c
   gcc xmul.o xdiv.o hello.o xsub.o xadd.o -o hello -Wall
   ```

   可以看出：`xmul.o`、 `xdiv.o`、 `xsub.o`、 `xadd.o`都是通过隐式规则生成的

2. 自定义模式规则

   针对隐式规则，若我们为目标文件自定义了规则，那么就不会调用隐式规则来生目标文件，而使用我们自定义的规则。打开1中的注释，再次make，输出如下：

   ```shell
   mxc@xxxxx:~/Projects/MakefileTest$ make
   gcc -c xmul.c -o xmul.o -Wall
   gcc -c xdiv.c -o xdiv.o -Wall
   gcc -c hello.c -o hello.o -I ./inc -Wall
   gcc -c xsub.c -o xsub.o -Wall
   gcc -c xadd.c -o xadd.o -Wall
   gcc xmul.o xdiv.o hello.o xsub.o xadd.o -o hello -Wall
   ```

   这里`xmul.o`、 `xdiv.o`、 `xsub.o`、 `xadd.o`都是通过自定义规则生成的

3. 静态模式规则

   指定某些目标文件由特定的规则生成，适用于生成同一类目标文件（如`.o`）的规则有多个情况。

   ```shell
   (a.o b.o):%.o:%.c
   	gcc -c $< -o $@
   (c.o d.o):%.o:%.c
   	gcc -c $< -o $@ -Wall
   ```

## 其他杂项

1. `ALL`

   默认情况下，`make`时，会将第一条规则中的目标当作最终目标文件，生成完最终文件就会结束`make`操作。可以在`makefile`文件中通过`ALL:最终目标`，来指定最终的目标。

2. `clean`

   可执行一些清理工作，如删除过程中生成的`.o`文件。

   ```makefile
   clean:
   	rm -rf a.o b.o a.out
   ```

   `make clean`命令用来执行makefile中的清理工作

   `make clean -n`命令用来查看makefile中清理工作将要清理的东西

3. `.PHONY`

   拿clean举例，如果make完成后，自己另外定义一个名叫clean的文件，再执行make clean时，将不会执行rm命令。 为了避免出现这个问题，需要`.PHONY: clean`，即无论clean文件是否存在，都执行rm命令。同理，ALL也需要如此处理，所以一般是`.PHONY:clean ALL`

4. 只编译指定目标

   若makefile一次make可编译出多个可执行文件，此时若想只编译其中某一个，只需要在执行make时，带上需要编译出的可执行文件名即可，如：`make copy`

## 示例1

```makefile
d_src = ./src
d_inc = ./inc
d_obj = ./obj

w_src = $(wildcard $(d_src)/*.c)
src = $(notdir $(w_src))
obj = $(patsubst %.c, $(d_obj)/%.o, $(src))

args = -g -Wall

#指定最终生成目标
ALL:hello

$(d_obj)/%.o:$(d_src)/%.c
    gcc -c $< -o $@ $(args) -I $(d_inc)
hello:$(obj)
    gcc $^ -o $@ $(args)

#清理目标文件
clean:
    rm -rf $(obj) hello
#伪目标
.PHONY:clean ALL
```

## 示例2

将目录下所有的`.c`分别编译为对应的可执行文件

```makefile
d_src = .

args = -g -Wall

src = $(wildcard $(d_src)/*.c)
exec = $(patsubst $(d_src)/%.c,$(d_src)/%, $(src));

ALL:$(exec)

$(d_src)/%:$(d_src)/%.c
    gcc $< -o $@ $(args)
clean:
    rm -rf $(exec)
.PHONY:ALL clean
```



