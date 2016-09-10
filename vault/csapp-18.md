title: 深入理解计算机系统 第 18 课 Dynamic Memory Allocation - Basic Concept
categories:
- Technique
toc: true
date: 2016-03-19 17:32:26
tags:
- 内存分配
- CMU
- 计算机
- 组成原理
---

前面了解了虚拟内存的相关知识，这节课我们来看看动态内存分配的基本概念，相信这之后就知道诸如 `malloc` 和 `new` 这类方法是怎么做的了。

<!-- more -->

---

## 基础概念

程序员通过动态内存分配（例如 `malloc`）来让程序在运行时得到虚拟内存。动态内存分配器会管理一个虚拟内存区域，称为堆(heap)，如下图所示：

![Dynamic Memory Allocation](/images/14584238419962.jpg)

分配器以 block 为单位来维护 heap，可以进行 allocate 或 free。有两种类型的分配器：

+ 显式分配器：应用分配并且回收空间（C 语言中的 `malloc` 和 `free`）
+ 隐式分配器：应用只负责分配，但是不负责回收（Java 中的垃圾收集）

## malloc

我们来看看 `malloc` 函数：

![The malloc Package](/images/14584242618725.jpg)

一个简单的例子：

```c
#include <stdio.h>
#include <stdlib.h>

void foo(int n) {
    int i, *p;
    
    /* Allocate a block of n ints */
    p = (int *) malloc(n * sizeof(int));
    if (p == NULL) {
        perror("malloc");
        exit(0);
    }
    
    /* Initialize allocated block */
    for (i=0; i<n; i++)
        p[i] = i;

    /* Return allocated block to the heap */
    free(p);
}
```

这节课中，为了讲述方便，我们做如下假设：

+ Memory is word addressed
+ Words are int-sized

![Assumptions Made in This Lecture](/images/14584252386542.jpg)

具体的例子：

![Allocation Example](/images/14584253826838.jpg)

程序可以用任意的顺序发送 `malloc` 和 `free` 请求，`free` 请求必须作用与已被分配的 block。

分配器有如下的限制：

+ 不能控制已分配 block 的数量和大小
+ 必须立即响应 `malloc` 请求（不能缓存或者给请求重新排序）
+ 必须在未分配的内存中分配
+ 不同的 block 需要对齐（32 位中 8 byte，64 位中 16 byte）
+ 只能操作和修改未分配的内存
+ 不能移动已分配的 block

## 性能指标

现在我们可以来看看如何去评测具体的分配算法了。假设给定一个 `malloc` 和 `free` 的请求的序列：

{% raw %} $$R_0, R_1, ..., R_k, ..., R_{n-1}$$ {% endraw %}

目标是尽可能提高吞吐量以及内存利用率（注意，这两个目标常常是冲突的）

吞吐量是在单位时间内完成的请求数量。假设在 10 秒中之内进行了 5000 次 `malloc` 和 5000 次 `free` 调用，那么吞吐量是 1000 operations/second

另外一个目标是 Peak Memory Utilization，就是最大的内存利用率，具体如下：

![Peak Memory Utilization](/images/14584265326302.jpg)

影响内存利用率的主要因素就是『内存碎片』，有两种类型：

+ internal fragmentation
+ external fragmentation

### 内部碎片

对于给定的 block，internal fragmentation 主要是因为 payload 小于 block size 造成的，如下图所示：

![internal fragmentation](/images/14584268109274.jpg)

主要是由以下原因导致；

+ Overhead of maintaining heap data structures
+ Padding for alignment purposes
+ Explicit policy decisions

只依赖于上一个请求的具体模式，所以比较容易测量。

### 外部碎片

指的是内存中没有足够的连续空间，如下图所示：

![External Fragmentation](/images/14584269418830.jpg)

依赖于未来的请求模式，所以比较难测量。

### 实现细节

在具体实现之前，需要考虑以下问题：

+ 给定一个指针，我们如何知道需要释放多少内存？
+ 如何记录未分配的 block ？
+ 实际需要的空间比未分配的空间要小的时候，剩下的空间怎么办？
+ 如果有多个区域满足条件，如何选择？
+ 释放空间之后如何进行记录？

一个标准的方式是在指针的前一个  word 中保存 block 的大小，通常称之为 header field 或 header，这种方式需要额外的一个 word，具体如下：

![Standard method](/images/14584273450722.jpg)

这里我们先给出常用的四种方式，这节课主要介绍第一种，下节课会介绍后面的方法：

![Keeping Track of Free Blocks](/images/14584273923045.jpg)


## 隐式 free 列表

对于每个 block 来说，我们需要知道大小和具体的状态（已分配/未分配），可以用两个 word 来存储，但是这样太浪费了。

如果一个 block 已经对其，低位地址一定是 0，所以我们可以用来当做 allocated/free 标志，当读入 word 大小的时候，需要标记出这个值。

![Implicit List](/images/14584304354717.jpg)

![Detailed Implicit Free List Example](/images/14584304662424.jpg)

寻找未分配的空间的方式如下，主要有三种：

![Finding a Free Block](/images/14584305912848.jpg)

确定空间之后，具体进行分配如下：

![Allocating in Free Block](/images/14584306987840.jpg)

```c
void addblock(ptr p, int len) {
    int newsize = ((len + 1) >> 1) << 1;  // round up to even
    int oldsize = *p & -2;                // mask out low bit
    *p = newsize | 1;                     // set new length
    if (newsize < oldsize)
        *(p+newsize) = oldsize - newsize; // set length in remaining
                                          // part of block
}
```

在释放空间的时候如果 block 后面也是未分配的空间，只做基本的处理的话，会出现有足够的位置，但是因为多余的分隔而找不到对应的位置，如下所示：

![Freeing a Block](/images/14584314075261.jpg)

解决的办法是 Coalescing：

![coalesce](/images/14584314805605.jpg)

另一种方法是双向 coalescing：

![Bidirectional Coalescing](/images/14584318651163.jpg)

具体 coalescing 的时候有四种情况：

![Constant Time Coalescing](/images/14584319803616.jpg)

下面是四种情况的：

Case 1：

![Case 1](/images/14584322811295.jpg)

Case 2：

![Case 2](/images/14584322969946.jpg)

Case 3：

![Case 3](/images/14584323407555.jpg)

Case 4：

![Case 4](/images/14584323554767.jpg)

Boundary Tags 的坏处就是会导致 internal fragmentation。

总结一下：

![Summary of Key Allocator Policies](/images/14584328927396.jpg)

最后是 implicit list 的总结：

![Implicit Lists: Summary](/images/14584329366933.jpg)

总体来说这一部分还是比较简单的，虽然简单，但是一定要理解清楚，因为下节课会介绍更加复杂的机制。

