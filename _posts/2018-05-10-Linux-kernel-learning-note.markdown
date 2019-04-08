---
layout: post
title: "Linux kernel learning note"
date: 2018-05-18 12:12 +0800
categories: linux
published: true
---

This is a brief learning note of `Understanding The Linux Kernel`

## Get started with kernel

obtain the kernel source

- using git
- installing the kernel
- using patches
- the kernel source tree
  - arch: architecture-specific source
  - block: block I/O layer
  - crypto: crypto API
  - Documentation: kernel source documentation
  - drivers: Device drivers
  - fs: the virtual file system and individual filesystems
  - init: kernel boot and initialization
  - IPC interprocess communication
  - kernel: core subsystems, such as the scheduler
  - mm: memory management system
  - sound: sound subsystem
  - virt: virtualazation infrastructure
  - net: networking subsystem
- building the kernel

  ```sh
  make config
  make menuconfig
  makegconfig
  make defconfig
  make oldconfig
  make
  make -jn
  ```

  - install the kernel

    on an x86 system using grub, your would copy `arch/i386/boot/bzImage` to `/boot`, name it something like `vmlinuz-version`, and edit `/boot/grub/grub/conf`, adding a new entry for the new kernel.

  - install modules

  ```sh
  # installing modules is automated and architecture-independent.
  make modules_install
  ```

- A beast of a different nature
  - the kernel has access to neither the C library nor the standard C header
  - the kernel is coded in GNU C
  - the kernel lacks the memory protection afforded to user-space
  - the kernel can not easily execute floating-point operations
  - the kernel has a small per-process fixed-size stack
  - because the kernel has asychronous interrupts, is preemptive, and supports SMP, synchronization and concurrency are major concerns within the kernel.
  - portability is important.
- no libc or standard headers
  - stdc is too large and too inefficient for the kernel. many of the usual libc functions are implemented inside the kernel.
- GNU C
  - the kernel is not programmed in strict ANSI C. the kernel developers make use of various language extensions available in gcc, they use both C99 and GNUC extensions to the C language.

## 1. Intruduction

## 2. memory addressing

- 2.1 memory address

  - logical addresss
    - Each logical address consists of segment and offset
  - linear address
  - physical address

  logical address --> segment unit --> linear address --> paging unit --> physical address

- 2.2 segmentation in hardware
  - real mode
  - protected mode
- 2.2.1 segmentation registers

  a logical address consists of two parts: a `segment identifier` and an `offset` that specifies the relative address within the segment.

  - segment idenfitier --> 16bits --> segmetn selector
  - offset --> 32bits

    segmentation registers

    - cs

      code segment register--> CPL current privilege level of the cpu

    - ss

      stack segment register

    - ds

      data segment register

- 2.2.2 Segment descriptors

  - GDT --> contain in main memory --> gdtr register
  - LDT --> contain in a process --> ldtr

    Each segment descriptor consists of the following fields:

    - a 32-bit based field that contains the linear address of the first byte of the segment

- 2.2.3 segment selectors
- 2.2.4 segmentation unit
- 2.3 segmentation in linux
- 2.4 paging hardware
- 2.4.1 regular paging
  - directory
  - table
  - offset
- 2.5.4 Process page tables

  the linear address space of a process is divided into two parts:

  - linear addresses from 0x00000000 to PAGE_OFFSET -1 can be addressed when the process is in either user mode or kernel mode
  - linear addresses from PAGE_OFFSET to 0xffffffff can be addresses only when the process is in kernel mode

## 3. process management

LKD:

- besides the executing program code(text section in Unix), Processes also include a set of resources:

  - open files
  - pending signals
  - internel kernel data
  - memory address space with one or more memory mappings
  - Thread(s) of execution
  - Data section containing global variables

- Threads of execution

  `Threads of exectuion`, often shortened to `threads`, are the objects of activity with the process.
  Each thread includes:

- program counter
- program stack
- set of processor registers

The kernel schedules individual threads, not process. Linux does not differentiate bwtween threads and processes. to linux a thread is just a special kind of process.

这一章主要讲解进程描述符以及相关数据结构， 进程切换，进程创建以及进程销毁

- 3.1 process descriptor and the task structure

  what each process is doing , process prority, whether it's runing or blocked, what address space has been assigned to it, what files it is allowed to address.

  The task_struct structure is allocated via the `slab allocator` to provide object reuse and cache coloring.defined in `<linux/sched.h>`

  ```c
  struct task_struct{
    volatile long state;/* -1 unrunnable, 0 runnable , > 0 stopped*/
    struct mm_struct *mm;
    int exit_state;
    int exit_code, exit_signal;
    pid_t pid;
    pid_t tgid;
    struct task_struct *real_parent;
    struct task_struct *parent;/* recipient of SIGCHLD, wait4() reports*/
    struct list_head children;
    struct list_head sibling;
    struct task_struct *group_leader;
    struct fs_struct *fs;
    struct files_struct *files;
    struct signal_struct *signal;
    struct sighand_struct *sighand;
  }
  struct thread_info{}
  ```

  Each task_struct (include/linux/sched.h) has:

- 3.1.1 `state`: `TASK_{RUNNING, INTERRUPTIBLE, UNINTERRUPTIBLE, STOPPED(reciving SIGSTOP, SIGSTP,ptrace), ZOMBIE}`
  - TASK_ZOMBIE: process execution is terminated, but the parent process has not yet issued a `wait()` like system call(`wait()`, `wiat3()`, `wait4()`, or `waitpid()`) to return information about the dead process. Before the wait()-like call is issued the kernel can not discard the data contained in the dead process descriptor because the parent process could need it.
- `fs_struct`: Filesystem information, current directory
- `files_struct`: open file information, pointers to file descriptors
- `mm_struct`: pointers to memory
- `signal_struct/sighand_struct`: signals received/signal handers
- 3.1.2 identifying a process
  process ID or (PID)
- 3.1.2.1 the task array

  The kernel reserves a global static array of size `NR_TASKS` called `task` in its own address space.

- 3.1.2.2 stroing a process descriptor

  since process are dynamic entities, process descriptors are stored in dynamic memory rather than in the memory area permanently assigned to the kernel. `Linux store two different data structures for each process in a single 8KB memory area: the process descriptor and the kernel mode process stack.`

- 3.1.2.3 the current macro

  process descriptors and stack cache: `free_task_struct`/ `alloc_task_struct`

- 3.1.2.4 process list:
  - `prev_task` `next_task`, the head of the list is init_task, process 0 or swapper.
  - `for_each_task`: sacns the whole process list
- 3.1.2.5 the list of TASK_RUNNING processes

  runqueue--> next_run, prev_run, add_to_runqueue(), del_from_runqueue(), wake_up_process()

- 3.1.2.6 The pidhash table and chained lists

Linux uses chaining to handle colliding PIDs

derive the process descriptor pointer corresponding to a PID, uses chaining to handle colliding PIDs

- 3.1.3 parenthood relationship among processes

  - parent: p_opptr(originall parent)
  - p_pptr(parent)
  - p_cptr(child)
  - p_ysptr(younger sibling)
  - p_osptr(older sibling)

- 3.1.4 wait queues

  Wait queues are implemented as cyclical lists whose elements include pointers to process descriptors. `void sleep_on(struct wait_queue **p)`

- 3.1.5 process usage limits:
  - RLIMIT_CPU
  - RLIMIT_FSIZE
  - RLIMIT_DATA
  - RLIMIT_STACK
  - RLIMIT_CORE
- 3.2 process context/process switching/

  - hardware contex
  - hardware support
  - linux code
  - saving the floating point registers

- 3.2.1 hardware context is stored in

  - TSS segment
  - Kernel Mode stack

  The set of data that must be loaded into the registers before the process resumes its execution on the CPU is called the hardware context.

  Process switching occurs only in Kernel Mode. The contents of all registers used by a process in User Mode have already been saved before performing process switching

- 3.2.3 the switch_to macro
- 3.2.4 saving floating point registers

- 3.3 process creation
- 3.3.1 the `clone()`, `fork()` and `vfork()` system calls

  lightweight processes are created in linux by using a function named `__clone()`-->parameters

  - fn
  - args
  - flags
    - signal number to be sent to the parent process when the child terminates: SIGCHLD
    - CLONE_VM|CLONE_FS|CLONE_FILES|CLONE_SIGHAND|CLONE_PID|CLONE_PTRACE|CLONE_VFORK
  - child stack

- clone--> do_fork() function --> alloc_task_struct

  do_fork()

  1. if the CLONE_PID flag has been specified, the do_fork()fucntion checks whether PID of the parent process is not null; if so , it reurns an error code. ONly the swapper process is allowed to set CLONE_PID; this is required when initializing a multiprocessor system
  2. the alloc_task_struct() function is invoked in order to get a new 8k union task_union memory area to store the process descriptor and the kernel mode stack of the new process
  3. the function follows the current pointer to obtain the parent process descriptor and copies it into the new process descriptor in the memory area just allocated.
  4. a few checks occur to make sure the use has the resources necessary to start a new process. First the function checks whether current->rlim[RLIMIT_NPROC].rlim_cur is smaller than or equal to the current number of processes owned by the user: if so, an error code is reutrned. The function gets the current number of processes owned by the uer from a per-user data structure named user_struct. this data structure can be found through a pointer in the user field os the process descriptor.
  5. the find_empty_process() function is invoked. if the owner of the parent process is not the superuser, this function checks whether nr_task (the total number of process in the system) is smaller than the NR_TASKS_LEFT_FOR_ROOT. If so, find_empty_process()involes get_free_takslot() to find a free entry in the task array. otherwise, it returns an error.
  6. the function writes the new process descriptor pointer into the previously obtained task entry and sets the tarray_ptr field of the process descriptor to the address of the entry
  7. if the parent process makes use of some kernel modules, th efunction increments the corresponding reference counters. Each kernel module has its own reference counter, which indicateds how many processes are using it. A module can not be removed unless its reference counter is null.
  8. updates some flags included in the flags field that have been copied from teh parent process
  9. the function invokes the get_pid() function to obtain a new PID, which will be assigned to the child process
  10. the function then update all the process descriptor fields that can not be inherited from the parent process, such as th fields that specify the process parenthood relationships.
  11. unless specified differently by the flags parameter, it involes copy_files(), copy_fs(), copy_sighand(), and copy_mm() to create new data structures and copy into them the values of the corresponding parent process data structures.
  12. invokes the copy_thread() to initialize the kernel mode stack of the child process with the values contained in the cpu registers when the clone() call was issued
  13. use the SET_LINKS macro to insert the new process descriptor in the process list
  14. hash_pid()
  15. increments the value of nr_tasks and current->user->count
  16. it sets the state field of the child process descriptor to TASK_RUNNING and then invokes wake_up_process() to insert the child in the runqueue list.
  17. if the CLONE_VFORK flag has been specified, the function suspens the parent process until the child releases its memory address space. in order to do this, the process descriptor includes a kernel semaphore called `vfork_sem`
  18. it returns the PID of the child, which will be eventually be read by the parent process in the user mode.

  - copy on write(COW) : allow both the parent and the child to read the same physical pages.
  - lightweight processes allow both parent and child to share many per-process kernel data structure, like paging tables and therefore the entire User mode address space and the open file tables.
  - the vfork system call creates a process that shares the memory address space of its parent.To prevent the parent from overwriting data needed by the child, the parent's execution is blocked until the child exits or execites a new program.`

- 3.3.2 kernel thread
  Kernel threads differ from regular processes in the following ways:
  - each kernel thread executes a single specific kernel function, while regular processes execute kernel functions only through system calls.
  - kernel threads run only in kernel mode, while regular processes run alternatively in kernel mode and in user mode.
  - since kernel threads run only in kernel mode, they use only linear addresses greater than `PAGE_OFFSET`, regular processes, on the other hand , use all 4 gigabytes of linear addresses, either in user mode or in kernel mode.
- 3.3.2.1 creating a kernel thread

  kernel_thread() function

- 3.3.2.2 Process 0

  the ancestor of all process, called process 0 or `the swapper process` is a kernel thread created from scratch during the initialization phase of linux by the `start_kernel()` function. This ancestor process makes use of the following data structures:

  - `a process descriptor` and `a kernel mode stack` stored in the `init_task_union` variable. The `init_task` and `init_stack` macro yield the addresses of the process descriptor and the stack, respectively.
  - the following tables, which the process descriptor points to:

    - init_mm
    - init_mmap
    - init_fs
    - init_files
    - init_signals

      the tables are initialized, respectively, by the following macros:

    - INIT_MM
    - INIT_MMAP
    - INIT_FS
    - INIT_FILES
    - INIT_SIGNALS

  - A TSS segment, initialized by the `INIT_TSS` macro
  - two segment descriptors, namely a `TSSD` and an `LDTD`, which are stored in the `GDT`
  - A `page global directory` stored in `swapper_pg_dir`, which may be considered as the kernel `page global directory` since it is used by all kernel threads

    the `start_kernel()` function initializes all the data structures needed by the kernel, enables interrupts, and creates another kernek thread, named `process 1`. more commonly referred to as the `init` process:

    ```c
    kernel_thread(init, NULL, CLONE_FS_CLONE_FILE|CLONE_SIGHAND)
    ```

    after having created the `init` process, process executes the `cpu_idele()` function, which essentially consists of repeatedly executing the `hlt` assembly language instruction with the interrupts enabled. Process is selected by teh scheduler only when there are no other processes in the `TASK_RUNNING` state.

- 3.3.2.3 Process 1

the kernel thead created by the process executes the init()function, which inturn invokes the `kernel_thread()` function four times to initiate four kernel threads needed for routine kernel tasks:

- `kflushd`: flush "dirty" buffers to disk to reclaim memory
- `kupdate`: flush old "dirty"buffers to disk to reduce risks of file system inconsistencies.
- `kpiod`: swaps out pages belonging to shred memory mappings.
- `kswapd`: performs memory reclaiming.

  then `init()` invokes the `execve()` system call to load the executable program init. As a result, the init kernel thread becomes a regular process having its own per-process kernel data structure.

- 3.4 destroy process

  - call exit
  - main returns
  - error

- 3.4.1 process termination
  all process terminations are handled by the `do_exit()` function, which removes most references to the terminationg process from kernel data structures. the do_exit() function executes the following actions:

  1. set the `PF_EXTING` flag in the flag field of the process descriptor to denote that the process is being eliminated.
  2. removes, if necessary, the process descriptor from an IPC semephore queue via the `sem_exit()` function or from a dynamic timer queue via the `del_timer()` function.
  3. Examines the process's data structures related to paging, filesystem, open file descriptors, and signal handing, respectively, with the `__exit_mm()`, `__exit_files()`, `__exit_fs()`and `__exit_sighand()` functions. These functions also remove any of these data structures if no other process is sharing it.
  4. set the state field field of the process descriptor to `TASK_ZOMBIE`.
  5. sets the `exit_code` field of the process descriptor to the process termination code. this value is either the `eixt()` system call parameter(normal termination), or an error code supplied by teh kernel(abnormal termination).
  6. invokes the`exit_notofy()` function to update the parenthood relationships of both the parent process and the children processes. All children process created by the terminating process become children of the `init` process.
  7. involes the `schedule()` function to select a new process to run. since a process in a `TASK_ZOMBIE` state is ignored by the scheduler, the process will stop executing right after `switch_to` macro in `schedule()` is invoked.

- 3.4.2 process removal
  - orphan process will be taken by the init process and will destroy the zombies through with a wait()-like system call.
  - the `release()` fucntion releases the process descriptor of `zombie process`

## 4. Interrupt

Intel 80x86 microprocessor manuals designate synchronous and asynchronous interrupts as exceptions and interrupts,

- synchronous interrupts : produced by the CPU control unit. Caused either by programming errors or by anomalous conditions that must be handled by the kernel.signals and page fault.
- asynchronous interrupts: generated by other hardware devices at arbitrary times with respect to the CPU clock signals. interval timers and I/O devices.

- difference between process switching and interrupt handling.

  the code executed by an interrupt or by an exception handler is not a process. Rather, it is a kernel control path that runs on behalf of the same process that was running when the interrupt occurred. As a kernel control path, the interrupt handler is lighter than a process.

- top half and bottom half.

  Interrupt can come at any time, when the kernel may want to finish something else it was trying to do.The kernel' goal is therefore to get the interrupt out of the way as soon as possible and defer as much processing as it can.

- Interrupt

  - maskable interrupts:
    - send to INTR pin
    - can be cleared with `IF` flag of `eflags` registers
    - All IRQs issued by I/O devices give rise to maskable interrupts.
  - nonmaskable interrupts:
    - send to NMI(nonmaskable interrupt) pin
    - can not be disabled by the IF flag.
    - only a few critical events, such as hardware failures, give rise to nonmaskable interrupts.

  hardware --> interrupt chip -->
  signal the CPU --> signal the OS

a key `difference between interrupt handling and process switching`: the code executed by an interrupt or by an exception handler is not a process. Rather, it is a kernel control path that runs on behalf of the same process that was running when the interrupt occurred.

- Interrupt and Exception vectors I

  Each interrupt or exception is identified by a number ranging from to 255;

  - 0-31 --> exceptions and nonmaskable interrupts.
  - 32-47 --> maskable interrupts--> interrupts caused by IRQs
  - 48 -255 used to identify software interrupts. linux use only 128(0x80 ) to implement system calls : user mode execute `int 0x80` --> switch to kernel mode and execute system_call()

- IRQs and Interrupts

  Each hardware device controller capable of issuing interupt requests has an output line designated as IRQ(Interrupt ReQuest). All existing IRQ lines are connected to the input pins of a hardware circut called the Interrupt Controller.Which performs the followingactions.

  - monitors the IRQ lines, checking for raised signals.
  - if a raised signal occurs on an IRQ line:

    - converts the raised signal received into a corresponding vector.
    - Stores the vector in and Interrupt Controller I/O port. thus allowing the CPU to read it via data bus.
    - sends a raised signal to the processor INTR pin -- that is, issues an interrupt.
    - waits until the CPU acknowledges that interrupt signal by writing into one of the Programmable Interrupt Controllers(PIC) I/O ports; when this occurs, clears the INTR line.

  - goes back to monitor

- exception

  the intel 80x86 issue roughly 20 different ecceptions. the kernel must provide a dedicated exception hadnler for each exception type. for some exceptions, the cpu control unit also generates a hardware error code and pusheds it in the kernel node stack before starting the exceprion handler.

  exception --> exception handler --> signal

  - 0: "divide error": (falut) divide_error(), SIGFPE
  - 3: "Breakpoint: (trap) caused by an int3(breakpoint) instruction(usually inserted by a a debugger)
  - 4: overflow
  - 14: "Page Fault": page_fault()
  - 16: floating point error

    Each device that generates interrupts has an associated interrupt handler.
    The interrupt handler for a device is part of the device's driver (the kernel code that manages the device).

    exception --> exception handler --> signal

interrupt context code executing in this context is unable to block

- interrupt descriptor table(IDT), idtr cpu register

  a system table called Interrupt Descriptor Table(IDT )associates each interrupt or exception vector with the address of the corresponding interrupt or exception hanler.

  - task gate--> linux does not use task gates
  - interrupt gate --> clears the IF flag, thus disabling further maksable interrupts.--> linux use it to handle interrupts.
  - trap gate --> dose not modifiy the IF flag --> linux use it to handle exceptions.

- Exception Handling

  - set_trap_gate
  - set_system_gate

- Interrupt vector (IRQ)

  - IRQ 0 timer interrupt
  - IRQ 1 keybord interrupt

- IRQ descriptors
  - do_IRQ
  - request_irq

/proc/interrupts
local_irq_disable
disable_irq
ret_from_sys_call() function
ret_from_exception() function

## 5. timer

init function when kernel start `time_init()`

- hardware clock

  - realtime clock
  - time stamp counter

- timer interrupt handler
  - update the time elaspsed since system startup: do_timer jiffies
  - udpate time and date:`update_times --> update_wall_time()`
  - determines how long the process has been running on the CPU and preempts it if it has exceeded the time allocated to it. `update_times--> update_process_times()-->need_resched--> schedule()`
  - updates resource usage statistics: `update_process_times`
  - check whether the interval of time associated with each software time has elasped; if so , invokes the proper function.

1> After system up, system time may be changed by NTP/GPS or other source, timeout which uses sem_timedwait will not be accurate, and from linux spec, there is no such sem_timedwait using monatomic timer.
2> system timer is not correct, previously use the realtime(wallclock) timer rather than the monatonic timer
check every 10ms, rather than 100ms, use nanosleep rather than sleep(sleep is not thread safe, not accurate)

xtime 是从 cmos 电路中取得的时间，一般是从某一历史时刻开始到现在的时间，也就是为了取得我们操作系统上显示的日期。这个就是所谓的“实时时钟”，它的精确度是微秒。The xtime variable of type struct timeval is where user programs get the current time and date.

jiffies 是记录着从电脑开机到现在总共的时钟中断次数。在 linux 内核中 jiffies 远比 xtime 重要

HZ and Jiffies

- the role of time

  - timeout

    implementing a timer is relatively easy: each timer contains a field that indicates how far in the future the timer should expire. This field is initially calculated by adding the right number of ticks to the current value of jiffies. The field does not change. Every time the kernel checks a timer, it compares the expiration field to the value of jiffies at the current moment, and the timer expires when jiffies is greater or equal to the stored value. This comparison is made via the time_after, time_before, time_after_eq, and time_before_eq macros, which take care of possible overflows of jiffies.

```txt
man sleep
DESCRIPTION
       sleep()  makes  the  calling  thread  sleep  until seconds seconds have
       elapsed or a signal arrives which is not ignored.
```

- static timer: `struct timer_struct---> run_old_timers()`
- dynamic time: `struct timer_list --> run_timer_list()`

## 6. Memory Management

Chapter talked how linux take advantage of Intel's segmentation and paging circuits to translate logical addresses into physical ones. In the same chapter, we mentioned that some portion of RAM is permanently assigned to the kernel and used to store both the kernel code and the static kernel data structures. This chapter talking about dynamic memory.

- 6.1/6.2 illustrate two different techniques for handling physically `contiguous memory`
- 6.3 illustrates a third technique that handles `noncontigunous memory`

- 6.1 page frame management
  - whey use 4k page frame size
    - the 4kb size is a multiple of most disk block sizes, so transfer of data between main memory and disk are more efficient.
    - By choosing a 4 KB allocation unit, the kernel can directly determine the memory allocation unit associated with the page where a page fault exception occurs

```c
typedef struct page {
    struct page *next;
    struct page *prev; //used to insert the descriptor into a doubly  linked  circular  list.
    struct inode *inode;
    atomic_t count; //set 0 to if the corresponding page frame is free. other if used by processes or kernel data structure
    unsigned long flags; //status of this page
}
```

- page/page fault

  - In terms of virtual memory, pages are the smallest unit.
  - most 32bit arch has 4KB sized page and 64 bits arch has 8KB sized page
  - page struct

- zones

  - zone dma
  - zone normal
  - zone highmem

- 6.1.1 getting page: alloc_page/free_page

- 6.1.2 the buddy system algorithm

the kernel must establish a robust and efficient strategy for allocating groups of contiguous page frames. In doing so , it must deal with a well-know memory management problem called external fragmentation: frequent requests and releases of groups of contiguous page frames are "scattered" inside blocks of allocated page frames. As a result, it may become impossible to allocate a large block of contiguous page frames. even if there are engouh free pages to satisfy the request.
strage:

- make use of the paging circuitry to mapp groups of noncontiguous free page frames into intervals of contiguous linear addresses
- develop a suitable technique to keep track of the existing blocks of free contiguous page frames, avoiding as much as possible the need to split a large free block in order to satisfy a request for a smaller one.

  - in some cases, contiguous page frames are really necessary, since contiguous linear addresses are not sufficient to satisfy the request. A typical example is a memory request for buffers to be assigned to a DMA processor. Since the DMA ignores the pading circuitry and accessses teh address bus directly while transferring several disk sectors in a single I/O operation, the buffers requested must be located in contiguous page frames.
  - Even if contiguous page frame allocation is not strictly necessary, it offers the big advantage of leaving the kernel paging table unchanged. What's wrong with modifying the page tables? frequent page table modifications lead to higher average memory access times, since they make the CPU flush the content of translation lookaside buffers.

the technique adopted by Linux to solve the external fragmentation problem is based on the well-know buffy system algorithm. All free page frames are grouped into 10 lists of blocks that contain groups of 1,2,4,8,16,32,64,128,256 ,and 512 contiguous page frames, respectively. The physical addresss of the first page frame of a block is a multiple of the group size: for example , the initial address of a 16-page-frame block is a multiple of 16 x 2^12.

- request: search --> look for larger--> split
- release: merge

- `mem_map` array
- An array having 10 elements of type `free_area_struct`, one element for each group size.---> `free_area[0]` and `free_area[1]`
- `bitmaps`

```c
struct free_area_struct {
    struct page *next;
    struct page *prev;
    unsigned int *map;
    unsigned long count;
};
```

- 6.2 memory area management

the buddy system algorithm adopts the page frame as the basic memory area. This is fine for dealing with relatively large memory requests, but how are we going to deal with requests for small memory areas, say a few tens or hundred of bytes.

- the slab allocator

the slab allocator groups objects into caches. Each cache is a "store" of objects of the same type. For instance, when a file is opened, the memory area needed to store the corresponding "open file" object is taken from a slab allocator cache named filp(for "file pointer"). the slab allocator caches used by Linux may be viewed at runtime by reading the `/proc/slabinfo` file

- 6.2.3 Slab Descriptor

  each slab of a cache has its own descriptor of type struct kmem_slab_s(equivalent to the type kem_slab_t)

- 6.2.13 general purpose objects

  infrequent request for memory areas are handled through a group of general caches whose objects have geometrically distributed sizes ranging from a minium of 32 to a maximum of 131072 bytes.

  objects of this type are obtained by invoking the `kmalloc` function

```c
void * kmalloc(size_t size, int flags)
```

- 6.3 Noncountiguous memory area management

- 6.3.2 Descriptors of Noncontiguous memory areas

```c
struct vm_struct{
    unsigned long flags;
    void * addr;
    unsigned long size;
    struct vm_struct *next;
}
```

- 6.3.3 Allocating a Noncontiguous Memory Area

the `vmalloc()` function allocates a noncontiguous memory area to the kernel. the parameter size denotes the size of the requested area. If the function is able to satisfy the request, then it returns the initial linear address of hte new area; otherwise, it returns a NULL pointer.

- get_vm_area
- 6.3.4 releasing a noncontiguous memory area

the `vfree()` function relaeses noncontiguous memory areas. Its parameter addr contains the initial linear addresss of the area to be released. vfree() fist scans the list pointerd by vmlist to finf the address of the area descriptor associated with the area to be released.

- kmalloc/kfree/vmalloc

a kernel function gets dynamic memory in a fairly straightforward manner by invoking one of a variety of functions:`__get_free_pages` from the buddy system algorithm, `kmem_cache_alloc()` or `kmalloc()` to use the slab allocator for specialized or general-purpose objects, and `vmalloc()` to get a noncontingunous memory area.

## 7. Process address space

A kernel function gets dynamic memory in a fairly straightforward manner by invoking one of a vaariety of functions: `_get_free_pages()` to get pages from the `buddy system algorithm`, `kmem_cache_alloc()` or `kmalloc()` to use the `slab allocator` for speciallized or general-purpose objects, and `vmalloc()` to get a noncontiguous memory area. If the request can be satisfied. each of these function returns a linear address identifying the beginning of the allocated dynamic memory arera.

- the kernel is the highest priority of the operating system: if some kernel function makes a request for dynamic memory, it must have some valid reason to issue that request and there is no point in trying to defer it.
- the kernel trusts itself: all kernel functions are assumed error-free, so it does not need to insert any protection against programming errors.

when allocating memory to user mode process, the situation is entirely different.

- process requests for dynamic memory are considered nonurgent. when a process's executable file is loaded, for instance, it is unlikely that the process will address all the pages of code in the near future. Similarly, when a process invokes `malloc()` to get additional dynamic memory it doesn't mean the process will soom access all additional memory obtained. So as a general rule, the kernel tries to defer allocating dynamic memory to user mode processes
- Since user programs cannot be trusted, the kernel must be prepared to catch all addressing errors caused by processes in user mode.

- 7.1 the process's address space

  the kernel represents intervals of linear addresses by means of resource called memory regions.

  typical situation in which a process gets new memory regions:

  - creates a new process to execute the command.
  - a running process may decide to load an entirely different program.
  - a running process may perform a "memory mapping" on a file
  - a process may keep adding data on its stack.
  - a process may create an IPC shared memory region to share data with other cooperating processes.
  - a process may expand its dynamic area(the heap) through a function such as `malloc()`

- system calls related to memory region creation and deletion

  - brk() change the heap size of the process
  - execve() loads a new executable file, thus changing the process address space
  - exit() terminates the current process and destroys its address spaces
  - fork() create a new process, and thus a new address space
  - mmap() creates a memory mapping for a file , thus enlarging the process address space
  - mumap() destroys a memory mapping for a file, thus contracting the process address space
  - shmat() create a shared memory region
  - shmdt() Destroys a shared memory region

- 7.2 the memory descriptor

  All information related to the process address space is included in a table referenced by the `mm` field of the process descriptor.

  ```c
  truct mm_struct {
      struct vm_area_struct *mmap; /* list of memory areas */
      struct rb_root mm_rb; /* red-black tree of VMAs */
      atomic_t count;
      pgd_t * pgd;
      void * segments;
      unsigned long rss;
      unsigned long total_cm; /* Total pages mapped */
      unsigned long task_size; /*size of  task space*/

      unsigned long start_code, end_code, start_data, end_data;
      unsigned long start_brk, brk, start_stack;
      unsigned long arg_start, arg_end, env_start, env_end;
    }
  ```

  - pgd, segments: point to the `page global directory` and `local descriptor table` of the processs.
  - rss: specifies the number of page frames allocated to the process.
  - count: denotes the number of processes that share the same struct `mm_struct` descriptor.

the `mm_alloc()` function is invoked to get a new memory descriptor. Since these descriptors are stored in a slab allocator cache, `mm_alloc()` calls `kmem_cache_alloc()`, initializes the new memory descriptor by duplicating the content of the memory descriptor of current, and sets the count field to 1.

- 7.3 Virtual Memory Areas/Memory Regions

  ```c
  struct vm_area_struct {
      struct mm_struct *vm_mm; /* The address space we belong to. */
      unsigned long vm_start; /*our start address within vm_mm*/
      unsigned long vm_end;/* The first byte after our end address within vm_mm. */
      struct vm_area_struct *vm_next, *vm_prev; /* linked list of VM areas per task, sorted by address */
  ```

  Each memory region descriptor identifies a linear address interval. The `vm_start` field contains the first linear address of the interval, while the `vm_end` field contains the first linear address outside of the interval; `vm_end - vm_start` thus denotes the length of the memory region. the `vm_mm` field points to the mm_struct memory descriptor of the process that owns the region.

- 7.3.1 memory region data structure -- AVL tree
- 7.3.2 memory region access rights
- 7.3.4 allocating a linear address interval

  the `do_map()` function creates and initializes a new memory region for the current process.

- `mmap()` and `do_mmap()`: Creating an Address Interval
- `munmap()` and `do_munmap()`: Removing an Address Interval
- page table
- 7.4 Page fault Exception handler
  - good_area
  - bad_area
  - no_context
- 7.4.1 handling a faulty address outside the addresss space -->bad_area/no_context --> `do_exit()`
- 7.4.2 handling a faulty address inside the address space --> good_area --> `handle_mm_fault()`
- 7.4.4 copy on write

- 7.5 creating and deleting a process address space
- 7.5.1 create a process address space
  - the kernel invokes the `copy_mm()` function while creating a new process
  - each process usually has its own address space, but lightweight processes can be created by callling `__clone()` with the `CLONE_VM` flag set. these share the same address space; that is, they are allowed to address the same set of pages.
- 7.5.2 deleting a process address space
  when a process terminates, the kernel invokes the `exit_mm()` function to release the addresss space owned by that process. Since the process is entering the `TASK_ZOMBIE` state, the function assigns the address space of the swapper process to it:

  the function then involes `mm_release()` and `mmput()` to release the process address space.

- 7.6 managing the heap

  Each unix process owns a specific memory region called heap, which is used to satisfy the process's dynamic memory requests. The `start_brk` and `brk` fiels of the memory descriptor delimit the starting and ending address, respectively, of that region. The following c library functions can be used by the process to request and release dynamic memory:

  - malloc(size)
  - calloc(n, size)
  - free(addr)
  - brk(addr)

        modify the size of the heap directly; the `addr` parameter specifies the new value of` current->mm->brk`, and the return value is the new ending address of the memory region(the process must check whether it coincides with the requested `addr` value).

    the `brk()` function differs from the other functions listed because it is `the only one implemented as a system call`: all the other functions are implemented in the C library by make use of `brk()` and `mmap()`.

    when a process in the user mode invokes the `brk()` system call, the kernel executes the `sys_brk(addr)` function. This function verifies first whether the addr parameter falls inside the memory region that contains the process code; if so, it reutrns immediately;

    if the process has asked to shrink the heap, `sys_brk()` invokes the `do_munmap()` function to do the job and then returns:

## 8. system calls

Putting an extra layer between the application and the hardware has several advantages.

- it makes programming easier, freeing users from studying low-level programming characteristics of harware devices.
- it greatly increases system security, since the kernel can check the correctness of the request at the interface level before attempting to satisfy it.
- the interfaces make programs more portable sice they can be compiled and executed correctly on any kernel that offerss the same set of interfaces.

- 8.1 api, POSIX, The C library
- 8.2 system call handlers and service routines

  in linux the system calls must be involed by executing the int \$0x80 assembly instruction. which raises the programmed exception having vector 128

  - system call numbers --> eax

- system call handler

  - saves the contents of most r egisters in the kernel mode stack(This operation is common to all system calls ans is coded in assembly language)
  - handles the system call by invoking a correcponding C function called the system call service routine
  - exits from the handler by meanss of the ret_from_sys_call() function(this function is coded in assembly language)

in order to associate each system call number with its corresponding service routine, the kernel makes use of a system call dispatch table; this table is stored i the sys_call_table array and has NR_syscalls entries(usually 256); the nth entry contains the service routine address of the system call having number n.

- 8.2.1 initializing system calls

  the trap_init() function invoked during kernel initialization sets up the `IDT` entry correcponding to vector 128 as follows

  ```c
  trap_init()
  set_system_gate(0x80, &system_call)
  ```

- system call implementation
- system call context

      the kernel is in process context during the execution of a system call. the `current` pointer points to the current task, which is the process that issued the syscall.

- 8.2.2 the `system_call()` function
  - saving the system call number and all the CPU registers
  - stores in `ebx` the address of the current process
  - check the paramter, number and value
- 8.2.3 parameter passing
- 8.2.4 verifing the parameter
- 8.2.5 accessing the process address space

```c
get_user()
put_user()
```

```c
SYSCALL_DEFINE0(vfork)
do_fork
copy_process
CLONE_NEWNS // new mount namespaces group
CLONE_FS //set if fs info shared between processes
CLONE_NEWUSER // new user namespace
dup_task_struct
free_task
cgroup_can_fork
copy_to_user copy_from_users
ptrace_event // SIGTRAP
ptrace_notify((event << 8) | SIGTRAP)

SYSCALL_DEFINE4
```

system call use the stack of the calling process just like a normal function call.
reference:

- [Unix system calls (1/2)](https://youtu.be/xHu7qI1gDPA?t=278)

## 9. Signals

- 9.1 The role of signals

  a signal is a very short message that may be sent to a process or to a group of processes. The only information given to the process is usually the number identifying the signal.

  signals serve two main purposes:

- to make a process aware that a specific event has occur
- to force a process to execute a signal handler function included in its code

  signals sent to a nonrunning process must be saved by the kernel until that process resumes execution.
  at anytime , only one pending signal of a given type may exist for a process. additional pending signal of the smae type to the same process are not queued but simply discarded.

- 9.1.1 actions performed upon receiving a signal

  there are three ways in which a process can respond to a signal

  - explicitly ignore the signal
  - execute the default action associated with the signal
  - catch the signal by invoking a correcponding signal-handler function

    the `SIGKILL` and `SIGSTOP` signal cannot be explicitly ignored or caught, and thus their default actions must always be executed. Therefore, `SIGKILL` and `SIGSTOP` allow a user with appropriate privilleges to destroy and top stop.

- 9.1.2 data structures associated with signals
- 9.1.3 operations on signal data structures
- 9.2 sending a signal
- 9.2.1 the `send_sig_info()` and `send_sig()` functions
- 9.2.2 the `force_sig_info()` and `force_sig()` functions
- 9.3 receiving a signal
- 9.3.1 ignoring the signal
- 9.3.2 executing the default action for the signal
- 9.3.3 catching the signal
  - `do_signal()`
  - `handle_signal()`
- 9.4 Real-time Signals
- 9.5 system calls related to signal handling
- 9.5.1 the kill() system call
- 9.5.2 changing a signal action
  - `sigaction(sig,act,oact)`

## 10. process scheduling

- 10.1 scheduling policy
  - interactive processes
  - batch proccesses
  - real-time processes
    system calls related to scheduling
  - nice()
  - getpriority()
  - setpriority()
    ...
- 10.1.1 process preemption

some real-time operating systems feature preempive kernels,which means that a process running in kernel mode can be interrupted after any instruction, just as it can in the user mode.the Linux kernel is not preemptive, which means that a process can be preemptived only while running in the user mode; nonpreemptive kernel design is much simpler, since most sychronization problems involving the kernel data structures are easily avoided.

- 10.2 the scheduling algorithm
  - static priority 0-99
  - dynamic priority
- 10.2.1 data structures used by the scheduler

  - need_resched
  - policy: SCHED_FIFO|SCHED_RR|SCHED_OTHER
  - rt_priority
  - priority
  - counter

- 10.2.2 the schedule() function
- multitasking
  - cooperative multitask
  - preemptive multitask

## 11. Kernel Synchronization Methods

- 11.2.2 atomic operation: `acomic_read`, `atomic_add` ,`atomic_set`
- 11.2.3 Interrupt diabling
- 11.2.4 Locking Through Kernel Semaphores

  ```c
  struct semaphore{}
  ```

## 12. The virtual file system

- regular files
- directories
- symbolic links

- 12.1 the role of Virtual file system.(VFS)

  The Virtual Filesystem (also known as Virtual Filesystem Switch or VFS) is a kernel software layer that handles all system calls related to a standard Unix filesystem. Its main strength is providing a common interface to several kinds of filesystems.

  filesystems supported by the VFS may be grouped into three main classes:

  - disk-based filesystems: ext4
  - network filesystems --> nfs smb
  - special filesystems (virtual filesystem): --> /proc, /dev/pts

- 12.1.1 the common file model

  - in the common file model each directory is regarded as a normal file, which contains a list of files and other directories
  - the Linux kernel cannot hardcode a particular function to handle an operation such as `read()` or `ioctl()`. Instead, it must use a `pointer` for each operation; the pointer is made to point to the proper function for the particular filesystem being accessed.
    file -> f_op-> read(...)
  - In short, the kernel is responsible for assigning the right set of pointers to the file variable associated with each open file, then for invoking the call specific to each filesystem that the f_op field points to.

    the common file model consists of the following object types:

  - the superblock object ---> filesystem control block
  - the inode object --> Stores general information about a specific file --> inode number
  - the file object
  - the dentry object

- 12.1.2 system calls handled by vfs

  - mount(), umount()
  - chmod()
  - ...

- 12.2.1 superblock objects
  super_operations
  - read_inode(inode)
  - write_inode(inode)
  - put_inode(inode)
  - delete_inode(inode)
- 12.2.2 inode objects
- 12.2.3 file objects
- 12.2.5 dentry objects
- 12.2.7 files associated with a process

- 12.3 filesystem mounting

- each file and directory in a partition is known by a unique inode number(root directory always has inode2)
- mkdir/rmdir/link/unlink/getdents system calll
- chdir/symlink

## 13. managing IO devides

- 13.1 I/O architecture
  - data bus
  - address bus
  - control bus
- 13.1.1 I/O ports
- 13.1.2 I/O interfaces
  - custom I/O interfaces
  - general-purpose I/O interfaces
- 13.1.2.1 Custom I/O interfaces
  - keyboard interface
  - Graphic interface
  - disk interface
  - network interface
  - bus mouse interface
- 13.1.2.2 general-purpose I/O interfaces
  - parallel port
  - serial port
  - universal serial bus(USB)
  - PCMCIA interface
  - SCSI(small computer system interface) interface
- 13.1.3 Device controller
- 13.1.4 Direct memory Access(DMA)

- 13.2 associting files with I/O devices
- 13.2.1 device files
  - type
  - major number
  - minor number
  - mknod() system call
- 13.2.1.1 block versue character devices
- 13.2.2 network cards

- 13.3 Kernel Modules/ Device driver
- 13.3.1 level of kernel support
- 13.3.2 monitoring I/O operations
- 13.3.2.1 polling mode
- 13.3.2.2 interrupt mode
- 13.4 character device handling
- 13.5 block device handling

some kernel module operation

```txt
lsmod: reading /proc/modules in fact, modules that is currently loaded into the kernel
insmod: insert a single module.
modprobe: insert a module including all other modules it depends on.
rmmod: remove a module.
modinfo: print some information about a module, e.g. author, description,parameters the module accepts, etc.
```

```sh
modinfo ahci (module information)
modprobe [-rv] nfs (loads kernel modules)
```

reference:

- [Listing Information about Loaded Modules](https://docs.oracle.com/cd/E52668_01/E54669/html/ol7-s1-modules.html)

## 14. disk caches

## 15. accessing regular files

- 15.1 reading and writing a regular file
- 15.2 memory mapping

## 16. swapping: methods for freeing memory

## 17. the ext2 filesystems

## 18. process comminications

- 18.1 pipes
- 18.2 fifos
- 18.3 system v ipc

## 19. program execution

- 19.1 executable files
- 19.1.4 program segments and process memory regions
- 19.2 execute formats

## A. system startup/Bootstrap

- the BIOS<-- ROM <-- RESET pin of the CPU, POST
- The bootloader <--MBR partion table
  - booting Linux from floppy disk
  - booting Linux from hard disk
- the setup_32() function
- the start_kernel() function

  - the page tables are initialized by invoking `paging_init()` function
  - the page descriptors are initialized by the `mem_init()` function
  - the final initialization of the IDT is performed by invoking `trap_init()` and `init_IRQ()`
  - the slab allocator is initialized by the `kmem_cache_init(`) and `kmem_cache_sizes_init()` functions
  - the system date and time are initialized by the `time_init()` function
  - the kernel thread for process 1 is created by invoking the `kernel_thread()` function. In turn, this kernel thead creates the other theads and executes the `/sbin/init` program/

- /UEFI/grub/lilo/systemd/init

  Kernel image/ramdisk
  compile the kernel
  kernel debug(Meng Ning's course)

  kenel startup grub booloader init/systemd

- [Linux Kernel 2.4 Internals](http://www.tldp.org/LDP/lki/index.html)

## funtion pointer

function is a block of code, the pointer point to the first instruction in the Text segment.

- Unlike normal pointers, a function pointer points to code, not data. Typically a function pointer stores the start of execute code.
- Unlike normal pointers, we `do not allocate de-allocate memory` using function pointers.
- A function's name can also be used to get function's address.
- Like normal data pointers, a function pointer can be passed as an argument and can also be returned from a function.
- Many object oriented features in C++ are implemented using function pointers in C. For example virtual functions. Class methods are another example implemented using function pointers.

reference:

- [function-pointer-in-c](https://www.geeksforgeeks.org/function-pointer-in-c/)
- [how-do-function-pointers-in-c-work](https://stackoverflow.com/questions/840501/how-do-function-pointers-in-c-work)

```c
#include <stdio.h>
int add(int a, int b)
{
    return a+b;
}
int main()
{
    int c;
    int (*p)(int, int);
    p = &add;
    c = (*p)(2,3);
    printf("%d",c);

}
```

## kernel data structures

- linked Lists

  - single and doubly linked lists
  - circular linked lists
  - moving through a linked list
  - manipulating linked lists

- queues
- maps
- binary trees

reference:

- [Understanding.Linux.Kernel](http://ermak.cs.nstu.ru/Understanding.Linux.Kernel.pdf)
- [理解进程创建、可执行文件的加载和进程执行进程切换，重点理解分析fork、execve和进程切换](https://blog.csdn.net/weixin_43389097/article/details/88743522)
- [跟踪分析Linux内核5.0系统调用处理过程](https://blog.csdn.net/Poor_Tony/article/details/88620626)