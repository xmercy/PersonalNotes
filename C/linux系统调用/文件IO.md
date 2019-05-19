# 文件IO

## 文件描述符（fd）

- PCB，进程控制块，是一个结构体。PCB内部有一个成员，文件描述符表，维护所有已打开文件的文件描述符。
- 已打开的文件底层对应一个结构体，文件描述符与该结构体对应。访问文件时，上层使用文件描述符，底层会根据文件描述符找到对应的结构体变量去操作文件。
- 文件描述符表默认情况下大小是1024，通过修改操作系统内核来更改。
- 已打开文件fd值可看作为在文件描述符表中的索引，该值一般从3开始，0对应`STDIN_FILENO`，1对应`STDOUT_FILENO`，2对应`STDERR_FILENO`。申请fd时，会从文件描述符表拿可用的、最小的索引号，赋给fd。

## open

```c
// open函数头文件
#include <unistd.h>
// O_WRONLY|O_CREAT头文件
#include <fcntl.h>
#include <stdio.h>
// errno头文件
#include <errno.h>
// strerror函数头文件
#include <string.h>

int main()
{
    int fd = open("./doc/doc1.txt", O_WRONLY|O_CREAT, 0644);
    // errno是系统变量，当程序出错时，errno变量值非0，正常情况是0
    printf("fd = %d, errno = %d:%s\n", fd, errno, strerror(errno));
    close(fd);
    return 0;
}
```

## read/write

```c
#include <unistd.h>
#include <fcntl.h>
#include <string.h>
#include <stdio.h>
#include <errno.h>
#include <stdlib.h>

#define BUF_SIZE 1024

int main(int args, char *argv[])
{
    if(args < 3)
    {
        printf("参数太少!\n");
        exit(EXIT_FAILURE);
    }
    
    // EXIT_SUCCESS 0, EXIT_FAILURE 1
    printf("success = %d, failure = %d\n", EXIT_SUCCESS, EXIT_FAILURE);

    // 打开文件
    int src_fd = open(argv[1], O_RDONLY);
    int tar_fd = open(argv[2], O_WRONLY|O_CREAT|O_APPEND, 0644);
    if(src_fd == -1 || tar_fd == -1)
    {
        perror("打开文件失败！");
        exit(EXIT_FAILURE);
    }
    // 复制文件
    char buf[BUF_SIZE] = { 0 };
    int r_len = 0;
    int w_len = 0;
    while((r_len = read(src_fd, buf, BUF_SIZE)) > 0)
    {
        w_len = write(tar_fd, buf, r_len);    
        if(w_len == -1)
        {
            perror("写入文件失败！");
            break;
        }   
    }
    if(r_len == -1)
    {
        perror("读取文件失败！");
        exit(EXIT_FAILURE);
    }
    // 关闭文件
    close(src_fd);
    close(tar_fd);
    return EXIT_SUCCESS;
}
```



使用putc与write（buf大小是1）拷贝相同文件，putc耗时更少情况分析

​	putc是库函数，一次向文件中写一个字节，底层最终也是调用write函数，write函数是系统调用提供的接口，buf大小设定为1是为了模拟与putc相同的场景。putc是对write的一次封装，理论上，拷贝相同的文件，write（buf大小是1）应该更快，但实际表现上，是putc更快。

​	原因分析：数据写入磁盘分为两个阶段，一阶段：数据从用户空间写到操作系统内核空间，二阶段：数据从内核空间到磁盘。一阶段数据由用户空间到内核空间耗时严重，write函数是数据从用户空间到内核空间的桥梁，因此使用write函数，且buf大小较小的情况下，拷贝文件很慢。前面提到putc底层也是通过write函数实现，为什么putc很快？原因是，putc这个函数内部维护了一个buf，默认情况下大小是4096字节，putc把写入的数据先写到buf，缓冲区满了再调用write函数，把数据从用户空间写到内核空间。调用putc相对于直接调用write函数，中间少了大量的数据从用户空间写到内核空间的过程，所以更快。

> 预读入与缓输出机制
>
> 前面说到，数据写入到磁盘分为：数据从用户空间到内核空间，再从内核空间到磁盘。内核空间底层同样维护了一个buf，默认大小4096字节，当buf满了以后，再把数据写入到磁盘，这个机制叫缓输出。当从磁盘读取数据的时候，数据先读取到内核空间的buf中，再从内核空间读取到用户空间，当读取的数据不足buf时，操作系统会预先读取buf大小的数据，这个机制叫预读入。

## lseek

`off_t lseek(int fd, off_t offset, int whence)`

参数：

1. fd：文件描述符
2. offset：偏移量
3. whence：偏移量相对于哪去偏移

返回值：

1. -1表示设置失败
2. 大于0表示光标偏移量，相对于文件起始位置。

应用场景：

1. 设置读取的位置
2. 可用于获取文件大小

`len = lseek(fd, 0, SEEK_END)`

3. 扩展文件大小，要先真正扩展，必须进行io操作

示例：

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <errno.h>
#include <fcntl.h>
#include <string.h>

int main(int args, char *argv[])
{
    char msg[] = "Hello World!\n";
    char buf;

    int fd = open("./doc/seek_test.txt", O_CREAT|O_RDWR,0644);
    if(fd == -1)
    {
        perror("打开文件失败！");
        return -1;
    }
    int w_len = write(fd, msg, strlen(msg));
    if(w_len == -1)
    {
        perror("写入文件失败！");
        return -1;
    }

    lseek(fd, 0, SEEK_SET);
    int r_len = 0;
    while((r_len = read(fd, &buf, 1)) > 0)
    {
        if(r_len == -1)
        {
            perror("读取文件失败！");
            return -1;
        }
        w_len = 0;
        w_len = write(STDOUT_FILENO, &buf, r_len);
        if(w_len == -1)
        {
            perror("写入文件失败！");
            return -1;
        }
    };

    close(fd);
    return 0;
}
```

## stat/lstat

`int stat(const char *pathname, struct stat *statbuf)`

`int lstat(const char *pathname, struct stat *statbuf)`

作用：两个函数都是获取文件属性信息，stat获取链接信息时，获取的是链接的实际文件信息，而lstat获取的是链接本身的信息

参数：

1. pathname：文件名
2. statbuf：封装文件信息并返回给上层

返回值：成功返回0，失败返回-1，并设置errno

举个栗子：

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/stat.h>

int main(int args, char *argv[])
{

    if(args < 2)
    {
        printf("参数太少！\n");
        exit(EXIT_FAILURE);
    }

    struct stat f_stat;
    int ret = stat(argv[1], &f_stat);
    if(ret == -1)\
    {
        perror("获取文件状态失败！");
        return -1;
    }

    printf("file size = %ld\n", f_stat.st_size);
    mode_t mode = f_stat.st_mode;
    printf("mode = %d\n", mode);
    exit(EXIT_SUCCESS);
}
```

## link/unlink

```c
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <unistd.h>

int main(int argc, char *argv[])
{
    if(argc != 3)
    {
        perror("参数数目不对！");
        exit(EXIT_FAILURE);
    }

    // 创建硬链接，文件名本身相当于硬链接
    link(argv[1], argv[2]);
    // 删除一个硬链接
    unlink(argv[1]);

    exit(EXIT_SUCCESS);
}
```

## opendir/readdir/closedir

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <dirent.h>
#include <errno.h>

int main(int argc, char *argv[])
{
    if(argc != 2)
    {
        printf("参数太少！\n");
        exit(EXIT_FAILURE);
    }

    DIR *p_dir = opendir(argv[1]);
    if(!p_dir)
    {
        perror("failed to open dir.");
        exit(EXIT_FAILURE);
    }

    struct dirent *p_dirent = NULL;
    // readdir之前，系统会先设置errno为0，出错时，设置为其他值，返回null
    // 读完文件夹下所有文件时，也会返回null，此时errno不变
    while((p_dirent = readdir(p_dir)))
    {
        if((!strcmp(p_dirent->d_name, ".")) ||(!strcmp(p_dirent->d_name, "..")))
            continue;
        printf("%ld\t%s\n", p_dirent->d_ino, p_dirent->d_name);
    }
    if(errno)
    {
        perror("failed to read dir.");
        exit(EXIT_FAILURE);
    }

    int ret = closedir(p_dir);
    if(ret == -1)
    {
        perror("failed to close dir.");
        exit(EXIT_FAILURE);
    }
    exit(EXIT_SUCCESS);
}
```

