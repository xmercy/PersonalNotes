# Linux命令

> 打个草稿，后续补充

## ls

`ls -lh`

## netstat

`netstat -tunlp`

`netstat -i` 查看网卡发送包、接收包、丢包信息

## grep

`grep "xxx" file1 > file2`

## ps

`ps -ef`

## unzip

## zip

## su

## sudo

## vim

## gdb

## wc

## df

`df -lhT`

## fdisk

`fdisk -l`

## parted

`parted -l`

## dstat

`dstat -lcdnrm`

## mkdir

`mkdir -p x1/x2/x3`创建文件夹，若中间某文件夹不存在，则自动创建

## scp

`scp -r sources/ root@192.168.3.141:/home/xc/src/kun-sql9/`

将文件夹sources及文件夹中的文件或子文件夹传输到远程目录/home/xc/src/kun-sql9/下

## find

`find path -name "filename"` 从指定路径下按名称查找文件

