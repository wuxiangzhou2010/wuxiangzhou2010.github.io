---
layout: post
title: "APUE learning note"
date: 2018-05-18 12:12 +0800
categories: linux
published: true
---

This is a brief learning note of apue

## Chapter 1. UNIX System Overview

This chapter gives basic Unix system concepts that are familiar to system administrators, from a programmer's perspective

## Chapter 2. UNIX Standardization and Implementations

This chapter discusses Unix standards, specifications and implementations

## Chapter 3. File I/O

This chapter discusses `unbuffered I/O`, which are not part of ISO C but are part of POSIX.1 and the Single UNIX Specification.

- file descriptors (Non-negative integer)
- functions:

  1. open and opennat
  2. creat
  3. close
  4. lseek (move the marker)
  5. read (with buffer in the OS for faster reading)
  6. write (with bufffer in the OS for faster writing)
  7. truncate (shrink and increase the file)

reference:

- [Unix system calls (2/2)](https://youtu.be/2DrjQBL5FMU?t=432)
- file sharing

  1. process table entry:

     - file descriptor flags(close on exec)
     - pointer to file table entry

  2. file table entry:
     kernel maintains a file table for all open files.
     - file status flags(O_RDONLY...)
     - current file offset
     - pointer to the v-node table entry for the file
  3. v-node structure:

     contains information about the type of the file and pointers to functions that operate on the file.

     This information is read from disk when the file is opened, so that all the pertinent information about the file is readily available.

     v-node also contains the i-node for the file.
     Linux has no v-node, instead, a generic i-node, Linux uses a file system-independent i-node and a file system-dependent i-node.

each process that opens the file gets its own file table entry, but only a single v-node table entry is required for a given file. One reason `each process gets its own file table entry is that each process has its own current offset for the file`.

it is possible for more than one file descriptor entry to point to the same file table entry:

1. dup:
2. fork: the parent and the child share the same file table entry for each open descriptor.

- pread pwrite:

seek and perform I/O atomically, equivalent to calling lseek and then read/write

- dup/ dup2

  new and old file descriptors `share the same file table entry`

- sync fsync fdatasync

  1. sync:queues all the modified block buffers for writing and returns. It `does not wait` for the disk writes to take place
  2. fsync: applies to a single file specified by the file descriptor fd, and `waits` for the disk writes to complete before returning.
  3. fdatasync: similar to fsync, but it affects only the data portions of a file

- fcntl

  can change the properties of a file that is already open

## Chapter 4. Files and Directories

This chapter centers on I/O for regular files

- file information: stat, fstat, fstatat, and lstat

  - stat: returns a structure of information about the `named file`
  - fstat: returns a structure of information about the given
    `file descriptor`

- file types: regular file, directory file,socket file, fifo file, symbolic link file
- set user id and set group id
- ownership and premission: access and faccessat, umask, chmod, fchmod, and fchmodat, chown, fchown, fchownat, and lchown,
- file system:
- link: link, linkat, unlink, unlinkat
- file times: futimens, utimensat
- reading directories:mkdir, mkdirat, rmdir, opendir, closedir, fdopendir, chdir, getdir

## Chapter 5. Standard I/O Library

- Buffering:

  - full buffered
  - line buffered
  - unbuffered.

    Standard error is always `unbuffered`.
    All other streams are `line buffered` if they refer to a terminal device; otherwise, they are fully buffered.

    setbuf, setvbuf

- open a stream: fopen, freopen, fdopen
- close a stream: flose
- reading and writing a stream:
  - character at a time I/O: getc, fgetc,gethchar
  - line at a time I/O:fgets, gets
  - binary I/O. fread, fwrite
- positioning a stream: ftell, fseek, rewind
- formatted I/O: fprintf

- umask - get/set default permissions for new files/directories

## Chapter 6. System Data Files and Information

- password, group, shadow file
- time and date

## Chapter 7. Process Environment

- main function
- Process Termination

  1. `exit`(总是执行 IO 库的清理关闭操作, return 0 相当于 exit(0))
  2. `_Exit` `_exit` (立即进入内核)
  3. `atExit`(登记函数结束后的清理过程， 跟注册的顺序相反。stack 的调用顺序)

- Command-Line Arguments

  ```c
  int main(int argc, char *argv[])
  ```

- Environment List

  ```c
  extern char **environ;
  ```

- Memory Layout of a C Program:

  - environ: 命令行参数和环境变量。

    环境表(字符串的指针数组)和环境字符串通常存放在进程存储空间的顶部(栈之上)。getenv 函数可以得到特定环境变量的值。

  - stack (栈区)
  - heap (堆区)
  - Uninitialized data segment:bss (未初始化的全局数据， 由 exec 初始化为 0)
  - Initialized data segment :初始化的全局数据
  - Text segment (正文程序代码区)

With Linux on a 32-bit Intel x86 processor, the text segment starts at location `0x08048000`

- Memory Allocation

  Memory can be allocated through many API calls:

  - malloc()
  - calloc()
  - realloc()
  - memalign()
  - posix_memalign()
  - valloc()
  - mmap()
  - brk() / sbrk()

  To return memory to the OS:

  - free()
  - munmap()

一般申请内存使用的是标准库提供的函数 malloc, cmalloc, remalloc.
这些函数和`系统调用`之间还隔着一个 `glibc 分配器`。

`malloc calloc realloc` ===> `glibc 分配器` ==> `brk/sbrk/mmap`

- heap 内存申请> 128k 使用 mmap, heap 顶不增加。
- < 128K 使用 brk, heap 顶增加。

1. malloc 下有一个将可用的内存块连接为一个长长的列表的所谓`空闲链表`
2. 沿着空闲链表找合适的块儿， 内存块一分为二， 一半分给用户， 一半挂到链表上。
3. 如果在空闲链表上找不到， 则由 glibc 发起系统调用(`sbrk, mmap`)

实际上 brk 和 sbrk 系统调用，就是调整 heap 顶地址指针。

- 当 glibc 发现堆顶有连续的 128k 的空间是空闲的时候，它就会通过 brk 或 sbrk 系统调用，来调整 heap 顶的位置，将占用的内存返回给系统。这时，内核会通过删除相应的线性区，来释放占用的物理内存。
- 内核只能通过调整堆顶指针的方式来调整调整程序占用的线性区， 只能通过调整线性区的方式来释放内存。
- 申请内存的时候，需要两个参数，一个是内存大小，一个是返回的指针；而释放内存的时候，却只要内存的指针： glibc 中，为每一块内存维护了一个 chunk 的结构。glibc 在分配内存时，glibc 先填写 chunk 结构中内存块的大小，然后是分配给进程的内存。 在进程释放内存时，只要指针-4 便可以找到该块内存的大小，从而释放掉
- 只要堆顶的部分申请内存还在占用，我在下面释放的内存再多，都不会被返回到系统中，仍然占用着物理内存。

  内存 debugger:

  - D.U.M.A. - Detect Unintended Memory Access
  - valgrind

reference:

- [malloc 函数工作机制](https://blog.csdn.net/nodeathphoenix/article/details/39339549)

- Environment variables
- setjmp and longjmp Functions
- 资源限制设置函数： getrlimit and setrlimit Functions

reference:

- [内核对内存的管理](https://blog.csdn.net/yang_yulei/article/details/24385573)
- [memory allocation method](https://en.wikibooks.org/wiki/Linux_Applications_Debugging_Techniques/Leaks)

## Chapter 8. Process Control

- process identifiers(进程标识符)

  process 0(swapper, part of the kernel, system process), 1(init/systemd, user space process and not system process in the kernel space)

  getpid/getppid/getuid/gettid

- fork funtion

  fork/vfork

  1. 父进程子进程并发执行
  2. 相互独立的地址空间(Copy-On-Write, COW)
  3. 共享文件：子进程继承了父进程所有的打开文件

  One characteristic of fork is that all file descriptors that are open in the parent are duplicated in the child, because it’s as if the dup function had been called for each descriptor. The parent and the child share a file table entry for every open descriptor.

  they share the same file offset.

  the `vfork` function was intended to create a new process for the purpose of e`xecuting a new program`

- 孤儿进程

  Orphan process (or orphaned child process) is any process whose `parent terminates`.init will take over the process. Whenever a process terminates, the kernel `goes through` all active processes to see whether the terminating process is the parent of any process that still exists. If so, the parent process ID of the surviving process is changed to be 1 (the process ID of init). This way, `it's guaranteed that every process has a parent`.

- 僵死进程。[即：已死，但无人收尸]

  Zombie process is a process that has terminated, but whose `parent has not yet waited for it`.

  防止僵死的方法： 1. fork 两次， 2. 父进程设置 SA_NOCLDWAIT

- [zombie process](https://www-cdf.fnal.gov/offline/UNIX_Concepts/concepts.zombies.txts)

- [daemon process](<https://en.wikipedia.org/wiki/Daemon_(computing)>)

  In multitasking computer operating systems, a daemon is a computer program that runs as a `background process`, rather than being under the direct control of an interactive user.

  a list of daemon process

  1. init
  2. crond
  3. dhdpd
  4. ftpd
  5. nfsd
  6. ntpd
  7. sshd
  8. httpd
  9. swapper
  10. systemd
  11. syslogd

  create a daemon process

  1. fork and parent process exit.
  2. child process setsid(), which create new session and become the process group leader
  3. set the root directory to `/`
  4. change umask to 0
  5. close file descriptors 0,1,2

- wait/waitpid/waitid/wait3/wait4 SIGCHLD

  When a process terminates, either normally or abnormally, the kernel notifies the parent by sending the `SIGCHLD` signal to the parent.

- exec 函数族:加载并运行程序 execl, execv

- Changing User IDs and Group IDs

  setuid/setgid/seteuid/setegid/setreuid/setregid

- Interpreter Files

  ```bash
  #!/bin/bash
  ```

- the system function

  ```c
  #include <stdlib.h>

  int system(const char *cmdstring);
  ```

- Process Scheduling

  - nice
  - getpriority

- Process Times: 进程时间

Three times can be measured:

    Wall clock time
    User CPU time
    System CPU time

reference:

- [UNIX 进程](https://blog.csdn.net/yang_yulei/article/details/17404021)

## Chapter 9. Process Relationships

1. Terminal login
2. Network login
3. Process group

   - getpgrp/getpgid/setpgid
   - Sessions:setsid/getsid

4. Job control
5. orphaned process group

## Chapter 10. Signals

- Signal dispositions

  1. ignore the signal
  2. catch the signal
  3. Let the default action apply

  The signals `SIGKILL` and `SIGSTOP` cannot be caught, blocked, or ignored.

SIGABRT abnormal termination(abort)
SIGALARM time expired(alarm)
SIGCHLD change in status of child
SIGPOLL pollable event(poll)
SIGKILL termination
SIGIO asynchronous i/o
SIGSEGV invalid memory reference

- kill
  send signal to the process, default signal is `SIGTERM(15) SIGKILL(9) SIGSEGV(11)`

        Signal     Value     Action   Comment
        ──────────────────────────────────────────────────────────────────────
        SIGHUP        1       Term    Hangup detected on controlling terminal
                                        or death of controlling process
        SIGINT        2       Term    Interrupt from keyboard
        SIGQUIT       3       Core    Quit from keyboard
        SIGILL        4       Core    Illegal Instruction
        SIGABRT       6       Core    Abort signal from abort(3)
        SIGFPE        8       Core    Floating point exception
        SIGKILL       9       Term    Kill signal
        SIGSEGV      11       Core    Invalid memory reference

  The signals SIGKILL and SIGSTOP cannot be caught, blocked, or ignored.

  ```sh
  kill -s SIGSEGV $PID  # this will generate a coredump file
  ```

- the core file
- signal function
- kill and raise
- alarm and pause
- sigset sigaction
- sleep, nanosleep, clock_nanosleep

reference:

- [how-to-handle-sigsegv-but-also-generate-core-dump](http://www.alexonlinux.com/how-to-handle-sigsegv-but-also-generate-core-dump)

## Chapter 11. Thread

- [UNIX 线程同步](https://blog.csdn.net/yang_yulei/article/details/18057203)
- [POSIX Threads Programming](https://computing.llnl.gov/tutorials/pthreads)

  线程同步的方式：互斥量 mutex, 读写锁 rwlock, 条件变量 condition variable

- thread concept
- thread identification
- thread creation

  ```c
  //create
  int pthread_create(pthread_t *thread, const pthread_attr_t *attr,
                          void *(*start_routine) (void *), void *arg);
  ```

- thread termination

  ```c
  //exit
  void pthread_exit(void *retval);
  int pthread_join(pthread_t thread, void **rval_ptr);

  //cancel
  int pthread_setcancelstate(int state, int *oldstate);//PTHREAD_CANCEL_ENABLE PTHREAD_CANCEL_DISABLE
  int pthread_setcanceltype(int type, int *oldtype);//asynchPTHREAD_CANCEL_DEFERRED  PTHREAD_CANCEL_ASYNCHRONOUS

  int pthread_cancel(pthread_t thread);
  ```

- Thread Synchronization

  ```c
    //normal lock
    pthread_mutex_unlock
    pthread_mutex_lock

    //read write lock
    int pthread_rwlock_rdlock(pthread_rwlock_t *rwlock);
    int pthread_rwlock_wrlock(pthread_rwlock_t *rwlock);
    int pthread_rwlock_unlock(pthread_rwlock_t *rwlock);

    //condition variables
    pthread_condattr_init
    pthread_condattr_destroy

    pthread_cond_wait
    pthread_cond_timedwait

    pthread_cond_signal
    pthread_cond_broadcast
    pthread_condattr_setclock

    //spin lock
    int pthread_spin_init(pthread_spinlock_t *lock, int pshared);
    int pthread_spin_destroy(pthread_spinlock_t *lock)
    int pthread_spin_lock(pthread_spinlock_t *lock);
    int pthread_spin_trylock(pthread_spinlock_t *lock);
    int pthread_spin_unlock(pthread_spinlock_t *lock);

  ```

  A spin lock is like a mutex, except that instead of blocking a process by sleeping, the process is blocked by busy-waiting (spinning) until the lock can be acquired.

- thread id/ attribute /stack

  ```c
  //attributes
  int pthread_attr_init(pthread_attr_t *attr);
  int pthread_attr_destroy(pthread_attr_t *attr);

  //Join state
  int pthread_attr_setdetachstate(pthread_attr_t *attr, int detachstate);
  int pthread_attr_getdetachstate(const pthread_attr_t *attr, int *detachstate);

  //shedule policy  SCHED_FIFO, SCHED_RR, and SCHED_OTHER
  int pthread_attr_setschedpolicy(pthread_attr_t *attr, int policy);
  int pthread_attr_getschedpolicy(const pthread_attr_t *attr, int *policy);
  // PTHREAD_INHERIT_SCHED：  using attr inherit scheduling attributes from the creating thread
  // PTHREAD_EXPLICIT_SCHED： Threads that are created using attr take their scheduling attributes from the values  specified  by the attributes object.
  int pthread_attr_setinheritsched(pthread_attr_t *attr,
                              int inheritsched);
  int pthread_attr_getinheritsched(const pthread_attr_t *attr,
                              int *inheritsched);
  //shedule parameter
  int pthread_attr_setschedparam(pthread_attr_t *attr, const struct sched_param *param);
  int pthread_attr_getschedparam(const pthread_attr_t *attr, struct sched_param *param);

  int pthread_setschedprio(pthread_t thread, int prio);

  // mutex
  int pthread_mutexattr_destroy(pthread_mutexattr_t *attr);
  int pthread_mutexattr_init(pthread_mutexattr_t *attr);

  pthread_mutexattr_setpshared
  pthread_mutexattr_destroy
  pthread_mutexattr_setprioceiling

  //stack， PTHREAD_STACK_MIN (16384)
  int pthread_attr_setstacksize(pthread_attr_t *attr, size_t stacksize);
      int pthread_attr_getstacksize(const pthread_attr_t *attr, size_t *stacksize);

  int pthread_attr_setstack(pthread_attr_t *attr,
                          void *stackaddr, size_t stacksize);
  int pthread_attr_getstack(const pthread_attr_t *attr,
                          void **stackaddr, size_t *stacksize);
  //the system allocates an additional region of at least guardsize bytes at the end of the thread's stack to act as the  guard  area  forthe stack (but see BUGS).
  int pthread_attr_setguardsize(pthread_attr_t *attr, size_t guardsize);
  int pthread_attr_getguardsize(const pthread_attr_t *attr, size_t *guardsize);

  // thread identifier
  pthread_t pthread_self(void);

  pthread_key_create

  int pthread_attr_setscope(pthread_attr_t *attr, int scope);
  int pthread_attr_getscope(const pthread_attr_t *attr, int *scope);
  PTHREAD_SCOPE_SYSTEM
  The thread competes for resources with all other threads in all processes on the system that are in the same scheduling allocation domain
  PTHREAD_SCOPE_PROCESS
  The thread competes for resources with all other threads in the same process that were also created with the PTHREAD_SCOPE_PROCESS contention scope.
  ```

  reference

  - [多进程 多线程](http://www.cnblogs.com/dongzhiquan/archive/2011/12/15/2289450.html)

## Chapter 12. Thread control

## Chapter 13. Daemon Process

- Error logging

## Chapter 14. Advanced I/O

- Nonblocking I/O
- Record locking: fcntl
- I/O multiplexing
  - select, pselect function
  - poll function
- asynchronous I/O

## Chapter 15. Interprocess Communication

- pipe fifo
- coprossses
- message queue
- semaphore
- shared memory

Approaches

    file
    signal
    Shared Memory  sys_shmget() sys_shmctl()
    message queue
    socket
    unix domain socket
    pipe
    named pipe

Synchronization

    Semaphores sys_semget() sys_semctl() sys_msgget() sys_msgctl()
    mutex
    lock/deadlock (Spinlocks)
    reader-writer locks

- semaphore

  link with `-pthread`

```c
#include <semaphore.h>

int sem_init(sem_t *sem, int pshared, unsigned int value);
int sem_destroy(sem_t *sem);
int sem_getvalue(sem_t *sem, int *sval);
int sem_post(sem_t *sem);
int sem_wait(sem_t *sem);
int sem_trywait(sem_t *sem);
int sem_timedwait(sem_t *sem, const struct timespec *abs_timeout);
```

condition variables

## Chapter 16. Network IPC: sockets

## Chapter 17. Advanced IPC
