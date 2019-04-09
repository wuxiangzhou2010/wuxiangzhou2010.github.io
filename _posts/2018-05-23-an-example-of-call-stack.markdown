---
layout: post
title: "An example of call stack"
date: 2018-05-23 12:12 +0800
categories: c
published: true
---

The C programming language pass parameter from Right-to-Left in a function call, support variable length of parameter, and the stack for new function is cleared by the caller when function returns.

This is a simple C program demonstrate how the stack works when perform a function call.

```c
// functiontest.c

int bar(int a, int b )
{
    return a +b;
}
int foo(int a, int b, int c)  
{
    int d;
    d =  bar(a ,b) +c;
    return d;
}


int main(void)
{
    int e;
    e = foo(1,2,3) ;
    return 0 ;
}
```

- compile the coding using gcc

```sh
gcc -S -m32 functiontest.c
grep  -E  -v '\t{0,}\.' functiontest.s
```

- [compile the code online](https://godbolt.org/)

note: use `gcc 7.3 x86-64` with `-m32 -O0` flag

```asm
fbar:
        pushl   %ebp
        movl    %esp, %ebp
        movl    8(%ebp), %edx
        movl    12(%ebp), %eax
        addl    %edx, %eax
        popl    %ebp
        ret
foo:
        pushl   %ebp
        movl    %esp, %ebp
        subl    $16, %esp
        pushl   12(%ebp)
        pushl   8(%ebp)
        call    bar
        addl    $8, %esp
        movl    %eax, %edx
        movl    16(%ebp), %eax
        addl    %edx, %eax
        movl    %eax, -4(%ebp)
        movl    -4(%ebp), %eax
        leave
        ret
main:
        pushl   %ebp
        movl    %esp, %ebp
        subl    $16, %esp
        pushl   $3
        pushl   $2
        pushl   $1
        call    foo
        addl    $12, %esp
        movl    %eax, -4(%ebp)
        movl    $0, %eax
        leave
        ret
```

Main:

1. 在 main 方法的调用栈中，将 foo 的参数`从右向左` 依次 push 到栈中(或者寄存器中)。
2. 把 main 方法当前指令的 下一条指令地址 （即 return address, EIP）push 到栈中。(隐藏在 call 指令中)
3. 使用 call 指令调用目标函数体 foo。(隐含将EIP 设置为call函数入口地址并执行)

Foo:

1. pushl ebp

   将 ebp 的当前值 pushl 到栈中，即保存 上一个栈帧的ebp。

2. movl esp,ebp

   将 esp 的值赋给 ebp，则意味着进入了 foo 方法的调用栈。

3. [可选]sub esp, XXX

   在栈上分配 XXX 字节的临时空间。（抬高栈顶）(编译器根据函数中的局部变量的总大小确定临时空间的大小)

4. [可选]pushl XXX

   保存（pushl）一些寄存器的值

而在 foo 方法调用完毕后，便执行前面阶段的逆操作：

1. 保存返回值

   通常将函数的返回值保存在寄存器`eax`中。

2. [可选]恢复(popl)一些寄存器的值。
3. movl esp,ebp

   恢复 esp 同时回收局部变量空间。（恢复原栈顶)

4. popl ebp

   将栈顶的值赋给 ebp，即恢复 main 调用栈的栈底。（恢复原栈底）

5. ret:

   从栈顶获得之前保留的 return address，并跳转到此位置继续执行。

其中3 和 4 可以用`leave`代替

- leave

    Releases the stack frame set up by an earlier `ENTER` instruction. The LEAVE instruction `copies the frame pointer (in the EBP register) into the stack pointer register (ESP)`, which releases the stack space allocated to the stack frame. The old frame pointer (the frame pointer for the calling procedure that was saved by the ENTER instruction) is then `popped from the stack into the EBP register`, restoring the calling procedure’s stack frame.

## 调用惯例

- 函数的传参数顺序和方式
- 栈的维护方式
- 名字修饰规则

- _cdecl
  
  CDeclaration的缩写，表示C语言默认的函数调用方法：所有参数从右到左依次入栈，这些参数由调用者清除，称为手动清栈。

reference:

- [call stack](https://blog.csdn.net/yang_yulei/article/details/45795591)
- [online c compiler and asembler](https://godbolt.org/)