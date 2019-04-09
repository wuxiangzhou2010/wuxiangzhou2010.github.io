---
layout: post
title: "Big Endian and little endian"
date: 2019-03-26 12:25 +0800
categories: c
published: true
---

- Big Endian

  store the most significant byte in the smallest address, naturally 0x1234 is a big Endian example

- Little Endian

  store the least significant byte inthe smallest address

![bigendianandlittleendian](/asserts/bigendianandlittleendian.png)

- `x86 系列 CPU 都是 little-endian 的字节序`.
- `网络字节顺序采用 big endian 排序方式`.

为了进行转换 bsd socket 提供了转换的函数 有下面四个

    htons 把 unsigned short 类型从主机序转换到网络序
    htonl 把 unsigned long 类型从主机序转换到网络序
    ntohs 把 unsigned short 类型从网络序转换到主机序
    ntohl 把 unsigned long 类型从网络序转换到主机序

s 就是 short, l 是 long, h 是 host, n 是 network

在使用 little endian 的系统中 这些函数会把字节序进行转换
在使用 big endian 类型的系统中 这些函数会定义成空宏

- 为什么会有小端字节序？

  一个不全面准确的答案是，计算机电路先处理低位字节，效率比较高，因为计算都是从低位开始的。所以，计算机的内部处理都是小端字节序。

- How to test big endian and little endian?

```c
#include <stdio.h>
// compile: gcc endian.c -o endian
// run : ./endian
// or use online c compiler: https://www.onlinegdb.com/online_c_compiler
int main()
{
    int i;
    char ch;
    // get size of type
    printf("1. sizeof int is %ld, sizeof char is %ld \n", sizeof(i),
           sizeof(ch));

    i = 0x1234;
    char *c = (char *)&i;

    printf("2. pointer of i  %p, pointer of c  %p \n", &i, c);
    printf("3. content of i  %p, content of *c  %p \n", i, *c);
    if (*c == 0x12)
    {
        printf("big endian\n");
    }
    else
    {
        printf("little endian\n");
    }
}
```

output:

```txt
1. sizeof int is 4, sizeof char is 1
2. pointer of i  0x7fff8dfb5564, pointer of c  0x7fff8dfb5564
3. content of i  0x1234, content of *c  0x34
little endian
```

reference:

- [Big and Little Endian](https://www.cs.umd.edu/class/sum2003/cmsc311/Notes/Data/endian.html)
- [endianness](https://blog.csdn.net/u011627161/article/details/70185406)
- [理解字节序](www.ruanyifeng.com/blog/2016/11/byte-order.html)

<!-- ## padding

align to the largest type in the struct -->
