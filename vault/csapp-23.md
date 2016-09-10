title: 深入理解计算机系统 第 23 课 Synchronization - Basics
categories:
- Technique
toc: true
date: 2016-04-07 11:30:32
tags:
- CMU
- 组成原理
- 计算机
- 同步
---

并行编程中最重要的问题就是同步，这节课我们来了解同步相关的基础知识

<!-- more -->

---

## 共享变量

在介绍同步之前，我们需要弄清楚一个定义，什么是 Shared variable（共享变量）？

> A variable `x` is *shared* if and only if multiple threads reference some instance of `x`

另外一个需要注意的是线程的内存模型，因为概念上的模型和实际的模型有一些差异，非常容易导致错误。

在概念上的模型中：

+ 多个线程在一个单独进程的上下文中运行
+ 每个线程有单独的线程上下文（线程 ID，栈，栈指针，PC，条件码，GP 寄存器）
+ 所有的线程共享剩下的进程上下文
    + Code, data, heap, and shared library segments of the process virtual address space
    + Open files and installed handlers

在实际的模型中，寄存器的值虽然是隔离且被保护的，但是在栈中的值并不是这样的（其他线程也可以访问）。

我们来看一个简单的例子：

![Example Program to Illustrate Sharing](/images/14600480676890.jpg)

这里有几个不同类型的变量，我们一一来看一下：

+ 全局变量：在函数外声明的变量
    + 虚拟内存中有全局唯一的一份实例
+ 局部变量：在函数内声明，且没有用 static 关键字
    + 每个线程的栈中都保存着对应线程的局部变量
+ 局部静态变量：在函数内用 static 关键字声明的变量
    + 虚拟内存中有全局唯一的一份实例

具体分析下上面例子中的变量，有：

![Mapping Variable Instances to Memory](/images/14600483444019.jpg)

具体来分析下，一个变量只有在被多个线程引用的时候才算是共享，在这个例子中，共享变量有 `ptr`, `cnt` 和 `msgs`；非共享变量有 `i` 和 `myid`。

![Shared Vairable Analysis](/images/14600486014966.jpg)

共享变量看起来不难，但是会导致一个并行编程中最重要的问题——同步问题。

## Critical Section

我们直接来看例子

![Improper Synchronization](/images/14600488439280.jpg)

为什么运行的时候，会出现不一样的结果呢？我们把操作 `cnt` 的部分抽出来单独看一看：

![Assembly Code for Counter Loop](/images/14600574108297.jpg)

这里有一点需要注意，`cnt` 使用了 `volatile` 关键字声明，意思是不要在寄存器中保存值，无论是读取还是写入，都要对内存操作（还记得 write-through 吗？）。这里把具体的步骤分成 5 步：HLUST，尤其要注意的 LUS 这三个操作，后面会继续说。

我们先来看看没有问题的情况：

![](/images/14600579848706.jpg)

这里我们可以看到，有颜色的指令连在一起，所以没有问题。但是一旦交叉，就有问题了，如：

![](/images/14600580383523.jpg)

## Progress Graph & Trajectory

为了描述上面这种情况，我们可以用 Progress Graph 来辅助，比如：

![Progress Graph](/images/14600581230106.jpg)

不同的轴代表在某线程中的指令执行顺序，每个点对应一个可能的执行状态，比如说图中的红点，表示线程 1 执行完了 L1，而线程 2 执行完了 S2。

Trajectory 的概念也很好理解，即可行的执行顺序，如下图：

![Trajectory](/images/14600582550787.jpg)

我们把 critical section 的边界画出来，就可以判断不同的 trajectory 是否是安全的了：

![](/images/14600583380649.jpg)


怎么样保证我们的执行不会走到不安全的区域里呢？有几种方法，这里我们只介绍 Semaphores。下面是比较常用的：

+ Semaphores - Edsger Dijkstra
+ Mutex & condition variables - Pthreads
+ Monitors - Java

## Semaphores

先看定义

> Semaphore: non-negative global integer synchronization vairable. Manipulated by P and V operations.

具体的操作为：

![](/images/14600591923220.jpg)

根据这样的设计，我们可以知道作为 Semaphore 变量其值一定是非负的。另外样例代码中已经封装了 Pthreads 的函数，如下：

![C Semaphore Operations](/images/14600599945369.jpg)

用法也很简单，在进入 critical section 之前，用 P 操作锁住，操作完成之后，用 V 才做释放。一些术语为：

![Terminology](/images/14600600984591.jpg)

就可以用 Semaphore 来保证前面的程序不会算错数了，但是因为要同步的缘故，速度会慢

![Proper Synchronization](/images/14600602184868.jpg)

具体的机制就是给不安全的区域上了个『锁』：

![Why Mutexes Work](/images/14600602784245.jpg)

## 总结

总结一下，在使用线程时，程序员脑中需要又一个清晰的分享变量的概念，共享变量需要互斥访问，而 Semaphores 是一个基础的机制。


