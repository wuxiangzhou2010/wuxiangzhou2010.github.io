---
layout: post
title: "A quick view of GDB"
date: 2019-04-09 12:12 +0800
categories: tools
published: true
---

## ptrace

不管远程调试还是本地调试都是使用的 `ptrace` 系统调用。

## gdb 三种调用方式：

1. `进入gdb, 然后输入 attach pid`. gdb 将对指定进程进行如下的操作：`ptrace(PTRACE_ATTACH,pid,0,0)`
2. `gdb ./a.out`。 当输入 run 的时候， 通过系统调用 fork 创建一个新进程， 在新创建的子进程中调用 `ptrace(PTRACE_TRACEME,0,0,0)`, 在子进程中调用 exec,加载用户指定的程序。
3. gdbserver, 远程调用。

## 断点原理：

1. ptrace 系统调用提供了一种方法，让父进程可以观察和控制其它进程的执行，检查和改变其核心映像及寄存器。主要用来实现断点调试和系统调用跟踪.
2. `信号`是实现断点的基础，当用 breakpoint 设置一个断点后，gdb 会找到该位置对应的具体地址，然后向该地址写入断点指令 `INT3`，即 0xCC. 目标程序运行到这条指令时，就会触发`SIGTRAP`信号，gdb 会首先捕获到这个信号。然后根据目标程序当前停止的位置在 gdb 维护的`断点链表`中查询，若存在，则可判定为命中断点。交付给目标程序的任何信号(除 SIGKILL 之外)都将被 gdb 先行截获.
3. gdb 暂停目标程序运行的方法是向其发送 `SIGSTOP` 信号。

## 调试命令

```sh
# 控制执行
run # 开始执行
next|n
#  next 指令可以实现单步调试，即每次只执行一行语句。
# 一行语句可能对应多条及其指令，当执行 next 指令时，
# gdb 会计算下一条语句对应的第一条指令的地址，然后控制目标程序走到该位置停止。

# 查看源码
list

# 设置 端点
break 22 # 22行
break main  # main 函数

# 查询断点/线程/参数/信号信息
info breakpoints
info thread
info locals
info args
info signals

# 查看backtrace
bt
bt full
where

# 查看栈帧信息
bt -n # 显示栈底的n个帧信息
bt n # 显示栈顶的n个帧信息

frame n # 选择帧号
up n #

thread thread# # 显示某个thread信息
# 在栈中向上移动n个帧

# print
x hexadecimal
a pointer
c character
s c string
t binary
u unsigned decimal

Examine memory: x/FMT ADDRESS.
ADDRESS is an expression for the memory address to examine.
FMT is a repeat count followed by a format letter and a size letter.
Format letters are o(octal), x(hex), d(decimal), u(unsigned decimal),
t(binary), f(float), a(address), i(instruction), c(char), s(string)
and z(hex, zero padded on the left

save breakpoints file-name-to-save
source file-name-to-save
tbreak (tb)
```

## 一个例子

```c
//1. a program demonstate gdb
// compile: gcc -g -o gdbtest gdbtest.c
// run : gdb gdbtest
#include <stdio.h>
int main()
{
    int foo[5], n;
    memset((char *)0x0, 1, 100);
    printf("Initial value of n is %d \n", n);
    return 0;
}
```

output:

```txt
Program received signal SIGSEGV, Segmentation fault.
__memset_avx2_unaligned_erms () at ../sysdeps/x86_64/multiarch/memset-vec-unaligned-erms.S:176
176	../sysdeps/x86_64/multiarch/memset-vec-unaligned-erms.S: No such file or directory.
(gdb) where
#0  __memset_avx2_unaligned_erms () at ../sysdeps/x86_64/multiarch/memset-vec-unaligned-erms.S:176
#1  0x0000555555554725 in main () at gdbtest.c:5
(gdb) bt
#0  __memset_avx2_unaligned_erms () at ../sysdeps/x86_64/multiarch/memset-vec-unaligned-erms.S:176
#1  0x0000555555554725 in main () at gdbtest.c:5
(gdb) info thread
  Id   Target Id         Frame
* 1    process 67467 "gdbtest" __memset_avx2_unaligned_erms () at ../sysdeps/x86_64/multiarch/memset-vec-unaligned-erms.S:176
(gdb)

```

- [use online debugger](https://www.onlinegdb.com/online_c_compiler)

reference:

- [gdb 的工作原理](https://blog.csdn.net/u012658346/article/details/51159971)
- [Step-by-step example for using GDB](https://kb.iu.edu/d/aqsy)
