title: 深入理解计算机系统 第 13 课 Exceptions and Processes
categories:
- Technique
toc: true
date: 2016-03-06 07:47:32
tags:
- 异常
- CMU
- 计算机
- 组成原理
---

了解完了链接，我们来看看在程序出错的时候会发生什么事情，这一课我们主要来了解异常与进程。

<!-- more -->

---

从开机到关机，CPU 做的工作其实很简单，就是不断读取并执行指令，每次执行一条，整个指令执行的序列，称为 CPU 的控制流。到目前为止，我们已经学过了两种改变控制流的方式：

+ 跳转和分支
+ 调用和返回

这对应于 program state 的改变。但是这实际上仅仅局限于程序的控制，没有办法去应对更加复杂的情况，比方说 system state 变化的时候：

+ 数据从磁盘或者网络适配器到达
+ 指令除以了零
+ 用户按下 ctrl+c
+ 系统的计时器到时间

所以我们这里会介绍另一种机制，称为 exceptional control flow。

## Exceptional Control Flow

Exceptional Control Flow 存在于系统的每个层级：

![](/images/14572937620861.jpg)

本节课我们先介绍前两种，下节课介绍后面两种。

## Exceptions

An **exception** is a transfer of control to the OS kernel in response to some event (i.e., change in processor state)

+ Kernel 是操作系统常驻内存的一部分
+ Event: Divide by 0, arithmetic overflow, page fault, I/O request completes, typing ctrl-c

具体的过程可以用下图表示：

![Exception 流程](/images/14572946534326.jpg)

系统会通过 Exception Table 来确定跳转的位置，每种事件都有对应的唯一的异常编号，发生对应异常时就会调用对应的异常处理代码：

![Exception Table](/images/14572947301161.jpg)

### Asynchronous Exceptions (Interrupts)

异步异常称之为中断，是有处理器外面发生的事情引起的，这种情况下：

+ 需要设置处理器的 interrupt pin
+ 处理完成后会返回之前控制流中的『下一条』指令

![中断的两个例子](/images/14572948483518.jpg)

### Synchronous Exceptions

同步异常是因为执行某条指令所导致的事件，分为 Traps, Faults 和 Aborts 三种情况：

![同步异常的三种类型](/images/14572949405706.jpg)

这里需要注意三种不同类型的处理方式，比方说 Traps 和中断一样，会返回执行『下一条』指令；而 Faults 会重新执行之前触发事件的指令；Aborts 则是直接退出当前的程序。

**System Call Example**

这里我们来了解一下系统调用 System Calls，系统调用看起来像是函数调用，但其实是走异常控制流的，在 x86-64 系统中，每个系统调用都有一个唯一的 ID，如：

![](/images/14572951617051.jpg)

而具体的的调用过程如下所示：

![](/images/14572951831475.jpg)


**Fault Example**

这里我们以 Page Fault 为例，来说明 Fault 的机制。Page Fault 发生的条件是：

+ 用户写入内存位置
+ 但该位置目前还不在内存中

比如：

```c
int a[1000];
main()
{
    a[500] = 13;
}
```

那么系统会通过 Page Fault 把对应的部分载入到内存中，然后重新执行赋值语句：

![](/images/14572953320001.jpg)

但是如果代码改为这样：

```c
int a[1000];
main()
{
    a[5000] = 13;
}
```

也就是引用非法地址的时候，整个流程就会变成：

![](/images/14572953822399.jpg)

具体来说会像用户进程发送 `SIGSEGV` 信号，用户进程会以 segmentation fault 的标记退出。

从上面我们就可以看到异常是非常底层的机制。

## Process 进程

> A process is an instance of a running program

进程是计算机科学中最为重要的思想之一，注意，和 "program" 或 "processor" 都不一样。

![进程示意图](/images/14572960158600.jpg)


进程给每个应用提供了两个非常关键的抽象：

![两个关键抽象](/images/14572959919795.jpg)

计算机会同时运行多个进程，比如说不同的前台应用，或者后台任务，比方说在 Mac 下输入 `top`，可以看到如下的进程信息：

![我的电脑当前的状态](/images/14572961362662.jpg)

具体的多进程模型如下所示：

![多进程模型（单核）](/images/14572962685162.jpg)

+ CPU 交替执行不同的进程
+ 虚拟内存系统会负责管理地址空间
+ 没有执行的进程的寄存器值会被保存在内存中

![切换到另一个进程执行，会载入原先的寄存器值(context switch)](/images/14572963666951.jpg)

而现代处理器一般有多个核心，所以可以真正同时执行多个进程：

![现代处理器执行模型](/images/14572964493864.jpg)

进程之间，也分并行与串行的关系：

+ Two processes run **concurrently** if their flows overlap in time
+ Otherwise, they are **sequential**

比方说下图中：

![](/images/14572965827499.jpg)

+ Concurrent: A&B, A&C
+ Sequential: B&C

不过在用户看来，执行的感觉是这样的：

![](/images/14572966466237.jpg)

### Context Switching

具体切换进程时，kernel 会负责具体的调度。

> The kernel is not a separate process, but rather runs as part of some existing process

控制流通过上下文切换的方式从一个进程到另一个进程，如下图所示：

![上下文切换 Context Switching](/images/14572967455790.jpg)

## Process Control 进程控制

### System Call Error Handling

在遇到错误的时候，Linux 系统级函数通常会返回 -1 并且设置 `errno` 这个全局变量来表示错误的原因。使用的时候记住两个规则：

1. You must check the return status of every system-level function
2. Only exception is the handful of functions that return void

例如，对于 `fork()` 函数，我们应该这么写：

```c
if ((pid = fork()) < 0) {
    fprintf(stderr, "fork error: %s\n", strerror(errno));
    exit(0);
}
```

如果觉得这样写太麻烦，可以利用一个辅助函数：

```c
void unix_error(char *msg) /* Unix-style error */
{
    fprintf(stderr, "%s: %s\n", msg, strerror(errno));
    exit(0);
}

// 上面的片段可以写为
if ((pid = fork()) < 0)
    unix_error("fork error");
```

我们甚至可以更进一步，把整个 `fork()` 包装起来，就可以自带错误处理，比如

```c
pid_t Fork(void)
{
    pid_t pid;
    if ((pid = fork()) < 0)
        unix_error("Fork error");
    return pid;
}
```

调用的时候直接使用 `pid = Fork();` 即可（注意这里是大写的 F）

### 获取进程信息

+ `pid_t getpid(void)` - 返回当前进程的 PID
+ `pid_t getppid(void)` - 返回当前进程的父进程的 PID

我们可以认为，进程有三个状态：

+ Running
    + 正在被执行、正在等待执行或者最终将会被执行
+ Stopped
    + 执行被挂起，在进一步通知前不会计划执行
+ Terminated
    + 进程被永久停止

**终止进程**

在下面三种情况时，进程会被终止：

1. 接收到一个终止信号
2. 返回到 `main` 
3. 调用了 `exit` 函数

![exit is called once but never returns](/images/14573038632263.jpg)

**创建进程**

调用 `fork` 来创造新进程

![fork is interesting becasue it is called once but returns twice](/images/14573039229956.jpg)

下面我们来看一个简单的例子：

![](/images/14573039725814.jpg)

有以下几点需要注意：

+ 调用一次，但是会有两个返回值
+ 并行执行，不能预计父进程和子进程的执行顺序
+ 拥有自己独立的地址空间（也就是变量都是独立的），除此之外其他都相同
+ 在父进程和子进程中 `stdout` 是一样的

### 进程图

进程图是一个很好的帮助我们理解进程执行的工具：

+ 每个节点代表一条执行的语句
+ a -> b 表示 a 在 b 前面执行
+ 边可以用当前变量的值来标记
+ `printf` 节点可以用输出来进行标记
+ 每个图由一个入度为 0 的节点作为起始

对于进程图来说，只要满足拓扑排序，就是可能的输出。我们还是用刚才的例子：

![](/images/14573044246995.jpg)

我们再来看三个稍微复制一点的例子：

![Two consecutive forks](/images/14573045555520.jpg)

![Nested forks in parent](/images/14573045862749.jpg)

![Nested forks in children](/images/14573046274270.jpg)

### Reaping Child Processes

即使进程已经终止，也还在消耗系统资源，我们称之为『僵尸』。为了『打僵尸』，就可以采用『收割』(Reaping) 的方法。父进程利用 `wait` 或 `waitpid` 回收已终止的子进程，然后给系统提供相关信息，kernel 就会把 zombie child process 给删除。

如果父进程不『收割』的话，通常来说会被 `init` 进程(pid == 1)回收，所以一般不必显式回收。但是在长期运行的进程中，就需要显式回收（例如 shell 和 server）。下面是几个僵尸进程的例子：

![这里，子进程可以成功被回收](/images/14573049855888.jpg)

![这里，因为子进程没有调用 `exit`，所以需要显式回收](/images/14573050208627.jpg)

**wait: Synchronizing with Children**

父进程通过调用 `wait` 函数来『收割』子进程

![](/images/14573055666401.jpg)

下面是一个具体的例子，同样用进程图来描述：

![wait: Synchronizing with Children](/images/14573056037360.jpg)

![If multiple children completed, will take in arbitrary order. Can use macros WIFEXITED and WEXITSTATUS to get information about exit status](/images/14573056447947.jpg)

**waitpid: Waiting for a Specific Process**

直接看例子：

![](/images/14573057138513.jpg)

**execve: Loading and Running Programs**

```c
int execve(char *filename, char *argv[], char *envp[])
```

具体的行为是：

![](/images/14573057856048.jpg)

为了理解 `execve` 的行为，我们要先理解程序在栈中的布局：

![栈的结构](/images/14573058581756.jpg)

一个具体的例子：

![execve 例子](/images/14573058863617.jpg)

## 总结

+ Exceptions
    + Events that require nonstandard control flow
    + Generated externally (interrupts) or internally (traps and faults)
+ Processes
    + At any given time, system has multiple active processes
    + Only one can execute at a time on a single core, though
    + Each process appears to have total control of processor + private memory space
+ Spawning processes
    + Call `fork`
    + One call, two returns
+ Process completion
    + Call `exit`
    + One call 
+ Reaping and waiting for processes
    + Call `wait` or `waitpid`
+ Loading and running programs
    + Call `execve`
    + One call, (normally) no return

