---
layout: post
title: "An example of call stack"
date: 2018-05-23 12:12 +0800
categories: c
published: true
---

The C programming language pass parameter from Right-to-Left, support variable length of parameter, and the stack for new function is cleared by the caller when function returns.

This is a simple C program demonstrate how the stack work when perform a function call.

```c
// functiontest.c
int foo(int a, int b, int c){}

int main(void)
{
    foo(1,2,3) ;
    return 0 ;
}
```

- compile the coding using gcc

```sh
gcc -S functiontest.c
```

- [compile the code online](https://godbolt.org/)

```asm
 foo:
     movl    %esp, %ebp
     nop
     popl    %ebp
     ret
 main:
     pushl   %ebp
     movl    %esp, %ebp
     pushl   $3
     pushl   $2
     pushl   $1
     call    foo
     addl    $12, %esp
     movl    $0, %eax
     leave
     ret

```

Main:

1、在 main 方法的调用栈中，将 foo 的参数从右向左 依次 push 到栈中。
2、把 main 方法当前指令的 下一条指令地址 （即 return address）push 到栈中。(隐藏在 call 指令中)
3、使用 call 指令调用目标函数体 foo。

Foo:

1. push ebp

   将 ebp 的当前值 push 到栈中，即保存 ebp。

2. mov ebp,esp

   将 esp 的值赋给 ebp，则意味着进入了 foo 方法的调用栈。

3. [可选]sub esp, XXX

   在栈上分配 XXX 字节的临时空间。（抬高栈顶）(编译器根据函数中的局部变量的总大小确定临时空间的大小)

4. [可选]push XXX

   保存（push）一些寄存器的值

而在 foo 方法调用完毕后，便执行前面阶段的逆操作：

1. 保存返回值

   通常将函数的返回值保存在寄存器`eax`中。

2. [可选]恢复(pop)一些寄存器的值。
3. mov esp,ebp

   恢复 esp 同时回收局部变量空间。（恢复原栈顶）

4. pop ebp

   将栈顶的值赋给 ebp，即恢复 main 调用栈的栈底。（恢复原栈底）

5. ret:

   从栈顶获得之前保留的 return address，并跳转到此位置继续执行。

reference:

- [call stack](https://blog.csdn.net/yang_yulei/article/details/45795591)
