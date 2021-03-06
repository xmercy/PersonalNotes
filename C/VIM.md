# VIM

## 快捷键及命令

### 底线命令模式

> 下面命令均是在命令模式前提下，输入特殊的字符进入底线命令模式，这里的特殊字符有`:`、`/`，下面的命令均带上了相应的特殊字符。

- `:vsp filename` 分隔出垂直vim窗口，用于编辑其他文件
- `:sp filename` 分隔出水平vim窗口，用于编辑其他文件
- `:!commond` 不退出vim情况下，执行shell命令

- `/string` 查找字符串`string`，进入查找后，按`n`向下查找，按`N`向上查找

- `:[range]s/from/to/[flags]` 
  1. s即substitude，该命令主要根据参数实现查找替换，from指定要匹配的字符串，可以是正则表达式，to指定替换后的字符串。
  2. range：`%`指匹配范围为所有行，`.`或者不指定range时，指匹配范围为当前行，`$`指匹配范围为最后一行，`1,$`指匹配范围为从第一行到最后一行，等价于`%`，`行号`指匹配范围为指定行号那一行
  3. flag：不指定则仅替换指定匹配范围中的第一个匹配到的字符串，`g`表示替换指定匹配范围中所有匹配到的字符串，`c`表示替换时需要用户进行确认，`n`统计匹配范围内匹配的字符串中数量，取消执行替换。flag可组合使用，常用组合有`gc`，`gn`
  4. 常用：`:%s/from/to/gc`、`:%s/from/to/gn`

`:f`查看当前文件名等信息

### 命令模式

`o` 在当前行下一行进入输入模式

`O` 在当前行上一行进入输入模式

`ctrl + w + h/j/k/l` vim多窗口间切换

`d + w` 删除一个单词

`d + d` 删除/剪切当前行

`x` 删除一个字符

`n + y + y` 复制n行

`v + h/j/k/l + y` 复制选中的字符串

`n + d + d` 删除/剪切n行

`p` 粘贴复制的内容

`ctrl + f` 向下翻页

`ctrl + b` 向上翻页

`G` 跳到文件尾

`g + g`跳到文件首行

`n + G` 跳到指定行

`h` 光标向右移动一个字符

`j` 光标向下移动一个字符

`k` 光标向上移动一个字符

`l` 光标向左移动一个字符

`^` 光标移动到行首（正则中表示开始位置）

`$` 光标移动到行尾（正则中表示结束位置）

`w` 光标移动到下一个单词词首

`e` 光标移动到下一个单词词尾

`b` 光标移动到上一个单词词首

`ge` 光标移动到上一个单词词尾

### 输入模式

nothing...

### 编码快捷键

`K` 在man手册查看函数的信息

`n + K`表示在man手册第n页查看函数的信息

## 配置VIM

