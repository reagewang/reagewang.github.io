---
layout: post
title: "基于POSIX mmap文件映射实现共享内存"
tags: [共享内存]
comments: true
---

write.c
```c++
#include <sys/mman.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

int main(int argc, char *argv[])
{
    int fd = shm_open("posixsm", O_CREAT | O_RDWR, 0666);
    ftruncate(fd, 0x400000);
    char *p = mmap(NULL, 0x400000, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
    memset(p, 'A', 0x400000);
    munmap(p, 0x400000);

    return 0;
}
```

read.c
```c++
#include <sys/mman.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>

int main(int argc, char *argv[])
{
    int fd = shm_open("posixsm", O_RDONLY, 0666);
    ftruncate(fd, 0x400000);
    char *p = mmap(NULL, 0x400000, PROT_READ, MAP_SHARED, fd, 0);
    printf("%c %c %c %c\n",p[0],p[1],p[2],p[3]);
    munmap(p, 0x400000);
    return 0;
}
```