title: 深入理解计算机系统 第 17 课 Virtual Memory - System
categories:
- Technique
toc: true
date: 2016-03-18 09:48:57
tags:
- 虚拟内存
- CMU
- 计算机
- 组成原理
---

了解了基本的虚拟内存概念，这节课我们来具体看看 Core i7 和 Linux 的内存系统，相信会对虚拟内存这一套机制有更深刻的认识。

<!-- more -->

---

开始之前我们还是先复习一下基本的概念：

![Review of Symbols](/images/14583183552533.jpg)

## 地址翻译实例

然后来看一个简单的例子：

+ 14 位的虚拟地址
+ 12 位的物理地址
+ page size 为 64 字节

如下图所示

![Addressing](/images/14583359973808.jpg)

TLB 的配置为：

+ 16 entries
+ 4-way associative

如下图所示

![TLB](/images/14583376303817.jpg)

然后来看看 page table，一共有 256 个 entry，这里列出前 16 个：

![Page Table](/images/14583376858566.jpg)

最后来看看系统本身缓存：

+ 16 lines, 4-byte block size
+ Physically addressed
+ Direct mapped

![Cache](/images/14583377349696.jpg)

一定要注意好不同部分的所代表的位置，这里我也会尽量写得清楚一些，来看第一个例子：

> 虚拟地址为 `0x03D4`

具体的转换过程如下图所示：

![第一个例子](/images/14583479163395.jpg)

具体来梳理一次：

先看 TLB 中有没有对应的条目，所以先看虚拟地址的第 6-13 位，在前面的 TLB 表中，根据 TLBI 为 3 这个信息，去看这个 set 中有没有 tag 为 3 的项目，发现有，并且对应的 PPN 是 0x0D，所以对应到物理地址，就是 PPN 加上虚拟地址的 0-5 位，而具体的物理地址又可以在缓存中找到（利用 cache memory 的机制），就可以获取到对应的数据了。

下面的例子同样可以按照这个方法来进行分析

![第二个例子](/images/14583493888490.jpg)

## 案例学习：Core i7

我们先来看看整体的架构

![Intel Core i7 Memory System](/images/14583495456299.jpg)

这里先留意一点，为什么 L1 d-cache 比上 L2 unified cache，和 L1 d-TLB 比上 L2 unified TLB 的比例一样呢？

![End-to-end Core i7 Address Translation](/images/14583500587332.jpg)

可以清楚地看到，从 TLB 到 Page table 以及对应 cache 的转换，结合上面的说明仔细体会下。接下来的内容比较偏理论，大家有一个基本的认识即可（因为平时编程理论上也不会涉及到这些）

Core i7 有 4 层的 page table，前 3 层的结构一样，如下图所示

![Core i7 Level 1-3 Page Table Entries](/images/14583503794880.jpg)

第四层有少许不同，具体看下面的说明：

![Core i7 Level 4 Page Table Entries](/images/14583505732854.jpg)

具体的翻译过程为：

![Core i7 Page Table Translation](/images/14583506241439.jpg)

接下来就可以回答前面的问题了，为啥是成比例的呢？原因很简单！用来加速 L1 的访问！

> Virtually indexed, physically tagged

![Cute Trick for Speeding Up L1 Access](/images/14583509113693.jpg)

## 案例学习：Linux Process

这一部分也是了解一下即可（具体可以不用太深究）

![Virtual Address Space of a Linux Process](/images/14583511838723.jpg)

Linux 以不同的『区域』来组织虚拟内存

![Linux Organizes VM as Collection of "Areas"](/images/14583514430157.jpg)

下面是处理 page fault 的机制：

![Linux Page Fault Handling](/images/14583519002966.jpg)


## 内存映射

初始化虚拟内存的过程，实际上就是把对应的虚拟内存和磁盘上的对象关联起来的过程，称之为内存映射(memory mapping)

![](/images/14583529040142.jpg)

Linux 中所谓的『交换分区』就是这么来的（我估计）

![Shared Objects](/images/14583530219645.jpg)

+ Process 1 maps the shared object
+ Process 2 maps the shared object
+ Two processes mapping a **private copy-on-write(COW)** object
+ Area flagged as private copy-on-write
+ PTEs in private areas are flagged as read-only

![Private Copy-on-write(COW) Objects](/images/14583530323902.jpg)

+ Instruction writing to private page triggers protection fault
+ Handler creates new R/W page
+ Instruction restarts upon handler return
+ Copying deferred as long as possible!

Fork 函数就是这种机制的一个很好的例子：

![The `fork` Function](/images/14583534506790.jpg)

接着来看看 `execve` 函数

![The `execve` Function](/images/14583535882515.jpg)

最后介绍了 `mmap` 函数，这里不赘述了。

这一部分比较偏向实际，主要还是要理解虚拟内存的机制（具体现代的处理器和操纵系统已经『太』复杂了）


