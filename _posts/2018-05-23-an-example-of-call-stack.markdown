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
int foo(int a, int b, int c)  
{
    int d;
    d =  a +b +c;
    return d;
}
int bar(int a, int b )
{
    return a +b;
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

```asm
foo(int, int, int):
  push ebp
  mov ebp, esp
  sub esp, 16
  mov edx, DWORD PTR [ebp+8]
  mov eax, DWORD PTR [ebp+12]
  add edx, eax
  mov eax, DWORD PTR [ebp+16]
  add eax, edx
  mov DWORD PTR [ebp-4], eax
  mov eax, DWORD PTR [ebp-4]
  leave
  ret
bar(int, int):
  push ebp
  mov ebp, esp
  mov edx, DWORD PTR [ebp+8]
  mov eax, DWORD PTR [ebp+12]
  add eax, edx
  pop ebp
  ret
main:
  push ebp
  mov ebp, esp
  sub esp, 16
  push 3
  push 2
  push 1
  call foo(int, int, int)
  add esp, 12
  mov DWORD PTR [ebp-4], eax
  mov eax, 0
  leave
  ret
```

Main:

1. 在 main 方法的调用栈中，将 foo 的参数`从右向左` 依次 push 到栈中(或者寄存器中)。
2. 把 main 方法当前指令的 下一条指令地址 （即 return address, EIP）push 到栈中。(隐藏在 call 指令中)
3. 使用 call 指令调用目标函数体 foo。(隐含将EIP 设置为call函数入口地址并执行)

Foo:

1. push ebp

   将 ebp 的当前值 push 到栈中，即保存 上一个栈帧的ebp。

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

   恢复 esp 同时回收局部变量空间。（恢复原栈顶)

4. pop ebp

   将栈顶的值赋给 ebp，即恢复 main 调用栈的栈底。（恢复原栈底）

5. ret:

   从栈顶获得之前保留的 return address，并跳转到此位置继续执行。


其中3 和 4 可以用`leave`代替

- leave

    Releases the stack frame set up by an earlier `ENTER` instruction. The LEAVE instruction `copies the frame pointer (in the EBP register) into the stack pointer register (ESP)`, which releases the stack space allocated to the stack frame. The old frame pointer (the frame pointer for the calling procedure that was saved by the ENTER instruction) is then `popped from the stack into the EBP register`, restoring the calling procedure’s stack frame.

- _cdecl
  
  CDeclaration的缩写，表示C语言默认的函数调用方法：所有参数从右到左依次入栈，这些参数由调用者清除，称为手动清栈。

reference:

- [call stack](https://blog.csdn.net/yang_yulei/article/details/45795591)
- [online c compiler and asembler](https://godbolt.org/)