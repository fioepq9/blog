---
title: '「6.S081」Lab1: Xv6 and Unix utilities'
date: 2022-04-21 08:34:05
tags:
  - 6.S081
  - 操作系统
categories:
  - 操作系统
  - 6.S081
---

## 环境安装

> 参考：[6.S081 / Fall 2019 (mit.edu)](https://pdos.csail.mit.edu/6.828/2019/tools.html)

### Installing on Debian

```bash
# 使用官方源
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get install git build-essential gdb-multiarch qemu-system-misc gcc-riscv64-linux-gnu binutils-riscv64-linux-gnu
# 安装python，以使用测试点
$ sudo apt-get install python-is-python3
```

### Testing your Installation

```bash
$ riscv64-unknown-elf-gcc --version
# riscv64-unknown-elf-gcc (GCC) 9.2.0 【Command Not Found？】
$ qemu-system-riscv64 --version
# QEMU emulator version 5.2.0 (Debian 1:5.2+dfsg-11+deb11u1)
# Copyright (c) 2003-2020 Fabrice Bellard and the QEMU Project developers
```

## 实验任务

### 启动xv6（Easy）

1. 获取xv6源码，并切换到util分支。

```bash
$ git clone git://g.csail.mit.edu/xv6-labs-2020
$ cd xv6-labs-2020
$ git checkout util
```

2. 构建并运行xv6。

```bash
$ make qemu
```

3. 退出xv6：键入`Ctrl + a`，然后键入`x`。

### sleep（Easy）

>  YOUR JOB
>
> 实现xv6的UNIX程序`sleep`：您的`sleep`应该暂停到用户指定的计时数。一个滴答(tick)是由xv6内核定义的时间概念，即来自定时器芯片的两个中断之间的时间。您的解决方案应该在文件*user/sleep.c*中

#### 实现过程

1. 编写*user/sleep.c*，如下：

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(2, "usage: sleep [number > 0]\n");
        exit(1);
    }
    printf("(nothing happens for a little while)\n");
    sleep(atoi(argv[1]));
    exit(0);
}
```

2. 在*Makefile*的`UPROGS`中添加`sleep`，如下：

```makefile
UPROGS=\
	...
	$U/_sleep\
```

#### 测试点

+ 运行测试：
```bash
$ ./grade-lab-util sleep
```

+ 结果：`OK`

![image-20220421104019486](https://raw.githubusercontent.com/ltlin9/note-img/master/img/2022/04/21/20220421-104028.png)

### pingpong（Easy）

> YOUR JOB
>
> 编写一个使用UNIX系统调用的程序来在两个进程之间“ping-pong”一个字节，请使用两个管道，每个方向一个。父进程应该向子进程发送一个字节；子进程应该打印“`<pid>: received ping`”，其中`<pid>`是进程ID，并在管道中写入字节发送给父进程，然后退出；父级应该从读取从子进程而来的字节，打印“`<pid>: received pong`”，然后退出。您的解决方案应该在文件*user/pingpong.c*中。

#### 实现过程

1. 编写*user/pingpong.c*，如下：

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"

int main(int argc, char *argv[]) {
    int fd[2];
    int status = pipe(fd);
    if (status != 0) {
        fprintf(2, "pipe open error.\n");
        exit(1);
    }
    int pid = fork();
    if (pid < 0) {
        fprintf(2, "fork error.\n");
        exit(1);
    }
    if (pid == 0) {
        // print
        printf("%d: received ping\n", getpid());
        // pipe write
        close(fd[0]);
        write(fd[1], "", 1);
        exit(0);
    }
    // pipe read
    close(fd[1]);
    read(fd[0], "", 1);
    // print
    printf("%d: received pong\n", getpid());
    exit(0);
}
```

2. 在*Makefile*的`UPROGS`中添加`pingpong`，如下：

```makefile
UPROGS=\
	...
	$U/_pingpong\
```

#### 测试点

+ 运行测试：

```bash
$ ./grade-lab-util pingpong
```

+ 测试结果：`OK`

![image-20220421110255674](https://raw.githubusercontent.com/ltlin9/note-img/master/img/2022/04/21/20220421-110300.png)

### Primes（Moderate/Hard）

> YOUR JOB
>
> 使用管道编写`prime sieve`(筛选素数)的并发版本。这个想法是由Unix管道的发明者Doug McIlroy提出的。请查看[this website](http://swtch.com/~rsc/thread/)，该网页中间的图片和周围的文字解释了如何做到这一点。您的解决方案应该在*user/primes.c*文件中。

#### 实现过程

1. 编写*user/primes.c*，如下：

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"

int fork_prime(int fd[2]) {
    int pid = fork();
    if (pid != 0) {
        return pid;
    }
    /*
     *  read first number(as prime) from left neighbor
     */
    close(fd[1]);
    // print prime
    int prime = -1;
    if (read(fd[0], &prime, sizeof(int)) <= 0) {
        exit(0);
    }
    printf("prime %d\n", prime);
    /*
     *  transfer to right neighbor
     */ 
    int fd_right[2];
    pipe(fd_right);
    fork_prime(fd_right);
    // write into pipe
    close(fd_right[0]);
    int num;
    while (read(fd[0], &num, sizeof(int)) > 0) {
        if (num % prime == 0) {
            continue;
        }
        write(fd_right[1], &num, sizeof(int));
    }
    close(fd_right[1]);
    // wait child process exit
    wait(0);
    exit(0);
}

int main(int argc, char *argv[]) {
    int fd[2];
    pipe(fd);
    fork_prime(fd);
    // write 2-35 into pipe
    close(fd[0]);
    for (int i = 2; i <= 35; i++) {
        write(fd[1], &i, 4);
    }
    close(fd[1]);
    // wait child process exit
    wait(0);
    exit(0);
}
```

2. 在*Makefile*的`UPROGS`中添加`primes`，如下：

```makefile
UPROGS=\
	...
	$U/_primes\
```

#### 测试点

+ 运行测试：

```bash
$ ./grade-lab-util primes
```

+ 结果：`OK`

![image-20220421125448687](https://raw.githubusercontent.com/ltlin9/note-img/master/img/2022/04/21/20220421-125453.png)

### find（Moderate）

> YOUR JOB
>
> 写一个简化版本的UNIX的`find`程序：查找目录树中具有特定名称的所有文件，你的解决方案应该放在*user/find.c*

#### 实现过程

1. 编写*user/find.c*，如下：

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"
#include "kernel/fs.h"

void find(const char* path, const char* filename) {
    char buf[512], *p;
    int fd;
    struct dirent de;
    struct stat st;

    if ((fd = open(path, 0)) < 0) {
        fprintf(2, "find: cannot open %s\n", path);
        return;
    }
    if (fstat(fd, &st) < 0) {
        fprintf(2, "find: cannot stat %s\n", path);
        close(fd);
        return;
    }
    if (st.type != T_DIR) {
        fprintf(2, "find: path error.\n");
        close(fd);
        return;
    }
    // path + '/' + dirname + '\0'
    if (strlen(path) + 1 + DIRSIZ + 1 > sizeof(buf)) {
        fprintf(2, "find: path too long\n");
        close(fd);
        return;
    }
    // buf = path + '/'
    strcpy(buf, path);
    p = buf + strlen(buf);
    *p++ = '/';
    // read file in dir
    while (read(fd, &de, sizeof(de)) == sizeof(de)) {
        if (de.inum == 0 || 
            strcmp(".", de.name) == 0 || 
            strcmp("..", de.name) == 0) {
            continue;
        }
        // let buf = path + '/' + filename + '\0'
        memmove(p, de.name, DIRSIZ);
        p[DIRSIZ] = 0;
        // get stat of file
        if (stat(buf, &st) < 0) {
            fprintf(2, "find: cannot stat %s\n", buf);
            continue;
        }
        // check
        if (st.type == T_FILE && strcmp(filename, de.name) == 0) {
            printf("%s\n", buf);
        }
        if (st.type == T_DIR) {
            find(buf, filename);
        }
    }
    close(fd);
    return;
}


int main(int argc, char *argv[]) {
    if (argc != 3) {
        fprintf(2, "usage error: find [path] [filename].\n");
        exit(1);
    }
    find(argv[1], argv[2]);
    exit(0);
}
```

2. 在*Makefile*的`UPROGS`中添加`find`，如下：

```makefile
UPROGS=\
	...
	$U/_find\
```

#### 测试点

1. 运行测试：

```bash
$ ./grade-lab-util find
```

2. 结果：`OK`

![image-20220421213849344](https://raw.githubusercontent.com/ltlin9/note-img/master/img/2022/04/21/20220421-213854.png)

### xargs（Moderate）

> YOUR JOB
>
> 编写一个简化版UNIX的`xargs`程序：它从标准输入中按行读取，并且为每一行执行一个命令，将行作为参数提供给命令。你的解决方案应该在*user/xargs.c*

#### 实现过程

1. 编写*user/xargs.c*，如下：

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "kernel/param.h"
#include "user/user.h"

int main(int argc, char *argv[]) {
    if (argc < 2) {
        fprintf(2, "usage error: xargs [command...]\n");
        exit(1);
    }
    if (argc == MAXARG) {
        fprintf(2, "usage error: too many args\n");
        exit(1);
    }
    char cmd[512], *p;
    p = cmd;
    char buf;
    while (read(0, &buf, sizeof(buf)) == sizeof(buf)) {
        if (buf == '\n') {
            *p = '\0';
            int pid = fork();
            if (pid < 0) {
                fprintf(2, "fork error.\n");
                exit(1);
            }
            if (pid == 0) {
                char **exec_argv = (char**)malloc(sizeof(char*) * (argc+1));
                for (int i = 1; i < argc; i++) {
                    exec_argv[i-1] = argv[i];
                }
                exec_argv[argc-1] = cmd;
                exec_argv[argc] = 0;
                exec(exec_argv[0], exec_argv);
                fprintf(2, "exec failed.\n");
                exit(1);
            }
            p = cmd;
            continue;
        }
        *p = buf;
        p++;
    }
    exit(0);
}
```




2. 在*Makefile*的`UPROGS`中添加`xargs`，如下：

```makefile
UPROGS=\
	...
	$U/_xargs\
```

#### 测试点

1. 运行测试：

```bash
$ make qemu
$ sh < xargstest.sh
# $ $ $ $ $ $ hello
# hello
# hello
# $ $

$ ./grade-lab-util xargs
```

2. 结果：`OK`

![image-20220421222211212](https://raw.githubusercontent.com/ltlin9/note-img/master/img/2022/04/21/20220421-222213.png)
