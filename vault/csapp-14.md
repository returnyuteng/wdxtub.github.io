title: 深入理解计算机系统 第 14 课 Signals and Nonlocal Jumps
categories:
- Technique
toc: true
date: 2016-03-06 07:47:36
tags:
- 异常
- CMU
- 计算机
- 组成原理
---

了解完了异常与进程，异常处理中另外两个很重要的部分是信号和非局部跳转，也就是我们这节课要介绍的内容。

<!-- more -->

---

Linux 的进程树，可以通过 `pstree` 命令查看，如下：

![Linux Process Hierarchy](/images/14573806185809.jpg)

我们以 shell 为例子，来看看整个过程是如何实现的：

![Shell: Execution is a sequence of read/evaluate steps](/images/14573806718703.jpg)

![Simple Shell eval Function](/images/14573807472000.jpg)

如果只有前台进程的话，我们的 shell 可以在前台工作完成之后进行回收。但是后台进程则会在终止之后成为僵尸进程，不会被回收并且造成内存泄露。

这怎么办呢？同样可以利用 Exceptional control flow，当后台进程完成时，内核会中断常规执行并通知我们，具体的通知机制就是『信号』(signal)。

## Signals

> A signal is a small message that notifies a process that an event of some type has occurred in the system

这样看来，其实是类似于 exception 和 interrupt 的，是由内核（在其他进程的请求下）向当前进程发出的。信号的类型由 1-30 的整数定义，信号所能携带的信息极少，一是对应的编号，二就是信号到达这个事实。下面是几个比较常用的信号的编号及简介：

![](/images/14573811635888.jpg)

> Kernel sends a signal to a destination process by updating some state in the context of the destination process

在下面两个场景中，内核会发送信号：

+ 内核检测到了如除以零(SIGFPE)或子进程终止(SIGCHLD)的系统事件
+ 另一个进程调用了 `kill` 指令来请求内核发送信号给指定的进程

> A destination process receives a signal when it is forced by the kernel to react in some way to the delivery of the signal

一个进程在接收到了信号之后，可以有几种不同的操作：

+ **忽略**这个型号
+ **终止**进程
+ **捕获**信号，通过执行 signal handler 完成（类似于异步中断中的 exception handler）

具体的过程如下：

![](/images/14573814877982.jpg)

> A signal is pending if sent but not yet received

同类型的信号至多只会有一个 pending signal，一定要注意这个特性，因为内部实现机制不可能提供较复杂的数据结构，所以信号的接收并不是一个队列。(If a process has a pending signal of type k, then subsequent signals of type k that are sent to that process are discarded)

一个 pending signal 至多只能被收到一次。

> A process can block the receipt of certain signals

被阻塞的信号仍然可以发送，但是直到不阻塞之后才能被接收

内核用 pending 位向量 和 blocked 位向量来维护每个进程的信号相关状态

+ pending: represents the set of pending signals
    + Kernel sets bit k in **pending** when a signal of type k is delivered
    + Kernel clears bit k in **pending** when a signal of type k is received 
+ blocked: represent the set of blocked signals
    + Can be set and cleared by using `sigprocmask` 函数
    + Also referred to as the **signal mask**

**进程组**

每个进程都只属于一个进程组，如下图所示：

![](/images/14573820359900.jpg)


我们可以据此指定一个进程组或者一个单独的进程，如：

![Sending Signals with `/bin/kill` Program](/images/14573821103841.jpg)

这里可以看到，第一个命令只会杀掉编号为 24818 的进程，但是第二个命令，因为有两个进程都属于进程组 24817，所以会杀掉进程组中的每个进程。

键盘同样可以让内核向每个前台进程发送 SIGINT(SIGTSTP) 信号

+ SIGINT - default action is to terminate each process
+ SIGTSTP - default action is to stop(suspend) each process

![Example of ctrl-c and ctrl-z](/images/14573823408948.jpg)

我们可以可以通过 `kill` 函数来发送信号：

![Sending Signals with kill Function](/images/14573824038965.jpg)

## 接收信号

所有的上下文切换都是通过调用某个 exception handler 完成的，内核会计算对易于某个进程 p 的 pnb 值：`pnb = pending & ~blocked`

+ 如果 `pnb == 0`
    + 那么就把控制交给进程 p 的逻辑流中的下一条指令
+ 否则
    + 选择 `pnb` 中最小的非零位 k，并强制进程 p 接收信号 k
    + 接收到信号之后，进程 p 会执行对应的动作
    + 对 `pnb` 中所有的非零位进行这个操作
    + 最后把控制交给进程 p 的逻辑流中的下一条指令

**默认动作**

每个信号类型都有一个预定义的『默认动作』，可能是以下的情况：

+ 终止进程
+ 终止进程并 dump core
+ 停止进程，收到 `SIGCONT` 信号之后重启
+ 忽略信号

`signal` 函数可以修改默认的动作：`handler_t *signal(int signum, handler_t *handler)`，具体来说：

![Different values for handler](/images/14573845678378.jpg)

我们再通过具体的代码来感受下：

![Signal Handling Example](/images/14573846209836.jpg)

可以这么理解 signal handler：

> A signal handler is a separate logical flow(not process) that runs concurrently with the main program

如下图所示：

![Signals Handlers as Concurrent Flows](/images/14573846875850.jpg)

![Another View of Signal Handlers as Concurrent Flows](/images/14573847143072.jpg)

还有一个需要注意的是，handler 也可以被其他的 handler 中断，控制流如下图所示：

![Nested Signal Handlers](/images/14573848399013.jpg)

## 阻塞/不阻塞信号

隐式的机制是，内核会阻塞与当前在处理的信号同类型的其他 pending signal，也就是说，一个 SIGINT handler 是不能被另一个 SIGINT 信号中断的。

如果想要显式阻塞，就需要使用 `sigprocmask` 函数了，以及其他一些辅助函数：

+ `sigemptyset` - Create empty set
+ `sigfillset` - Add every signal number to set
+ `sigaddset` - Add signal number to set
+ `sigdelset` - Delete signal number from set

我们可以用下面这段代码来临时阻塞特定的信号：

![Temporarily Blocking Signals](/images/14573850645959.jpg)

## 安全地处理信号

> Handlers are tricky because they are concurrent with main program and share the same global data structures

尤其要注意因为并行访问可能导致的数据损坏的问题，这里提供一些基本的指南（后面的课程会详细介绍）

![Guidelines for Writing Safe Handlers](/images/14573852332585.jpg)

另外一个需要注意的问题是 Async-Signal-Safety。

> Function is async-signal-safe if either reentrant(all variables sotred on stack frame) or non-interruptible by signals

Posix 标准指定了 117 个 async-signal-safe 的函数（可以通过 `man 7 signal` 查看）

![许多常用的函数都不是 async-signal-safe 的](/images/14573853746526.jpg)

因为输出函数不是 async-signal-safe 的，所以最好使用课本中提供的 `csapp.c` 中的相关 handler

+ `ssize_t sio_puts(char s[])` - Put string
+ `ssize_t sio_putl(long v)` - Put long
+ `void sio_error(char s[])` - Put msg & exit

![](/images/14573854667518.jpg)

正确的信号处理方法：

+ You can't use signals to count events, such as children terminating
+ Must wait for all terminated child processes.

![Put `wait` in a loop to reap all terminated children](/images/14573856728934.jpg)

还有一个问题，不同 Unix 版本有不同的 signal handling semantics，我们给出的解决方案是使用 `sigaction`，如下：

![Portable Signal Handling](/images/14573860066787.jpg)

## 避免进程竞争

我们之前的 shell 代码会出现微妙的同步错误，因为我们假设父进程会在子进程之前执行，代码如下：

![](/images/14573866195444.jpg)

![](/images/14573866343912.jpg)

我们需要在循环中添加同步条件，确保父进程和子进程的顺序（注意比较）

![Corrected Shell Program without Race](/images/14573867961969.jpg)

## 显式等待信号

我们也可以用类似与等待前台任务执行的方式来等待子进程，方法如下：

![Handlers for program explicitly waiting for SIGCHLD to arrive](/images/14573869619954.jpg)

![Handlers for program explicitly waiting for SIGCHLD to arrive](/images/14573869910604.jpg)

这里的代码是正确的，但是我们注意 `while(!pid)` 这一句，通过忙等待的方式实现同步，非常浪费资源，而其他方式看起来也不行：

![Other options](/images/14573870533166.jpg)

怎么办呢？我们的解决办法是 `sigsuspend`，函数为：

`int sigsuspend(const sigset_t *mask)`

等同于 atomic 版本的：

![](/images/14573871178306.jpg)

所以代码如下：

![Wating for Signals with sigsuspend](/images/14573871346551.jpg)

## Nonlocal Jump

这一部分比较简单，主要是使用 `setjmp` 与 `longjmp`

![NoNonlocal Jumps: setjmp/longjmp](/images/14573872756594.jpg)

![NoNonlocal Jumps: setjmp/longjmp](/images/14573873026532.jpg)

我们可以利用这种方式，来跳转到其他的栈帧中，比方说在嵌套函数中，我们可以利用这个快速返回栈底的函数：

![](/images/14573874602644.jpg)

但是也有限制，必须在栈中（也就是还没完成）才可以进行跳转，下面的例子中，因为 P2 已经返回，所以不能跳转了：

![](/images/14573875130970.jpg)

最后是一个非常清晰的例子：

![A Program that restarts itself when ctrl-c](/images/14573875598038.jpg)

## 总结

这两个基本说完了 exceptional control flow 的全部内容，可能会稍微有点难以理解，我会在后面的习题课中尽可能详细说明。


