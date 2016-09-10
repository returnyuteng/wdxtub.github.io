title: How to Write Fast Code 第 1 课 背景知识
categories:
- Technique
date: 2016-02-01 09:51:22
tags:
- CMU
- 编程
- 架构
- 并行
---

这一课主要介绍并行编程的背景知识和一些基本的技巧，给大家一个整体的认知。

<!-- more -->

---

简单来说，提高程序运行速度的方式有很多种，除了算法优化，还有其他许多软件硬件相结合的技巧。不同层级上的优化，带来的收益也会不一样，比方说在软件架构上的优化，可能可以带来 20-100 倍的收益，算法层面上的优化是 10-40 倍，而数据结构上的优化是 1.5-8 倍（当然这些数字都不是绝对的）

这门课主要涉及 OpenMP，CUDA 和 Hadoop（也会有一些 Spark 的内容，机器学习的部分还在商量中）。我主要负责 OpenMP 和 Hadoop 的部分。

具体需要理解的概念，基本上在第 0 课中我的笔记都有涉及，课件里的例子一定要重点掌握，比如：

+ Instruction level parallelism 的原理和机制
+ SIMD 的原理和机制（SSE 的用法）
+ Simultaneous Multithreading(SMT)的概念
+ Memory Hierarchy 的原理及如何进行利用（矩阵相乘的例子）
+ Compulsory miss / Capacity / Conflict

接着是两幅图：Platform + Technique

![](/images/14543452357180.jpg)

![](/images/14543452584935.jpg)

最后说一下 Kmeans 这个算法，具体的算法过程这里不赘述了，基本的过程参见下图：

![](/images/14543458160793.jpg)

如果想要理解更清楚一些，还需要去看看 EM 算法（这里同样不写）。然后对应到具体的代码，看看有哪些地方可以并行，哪些地方不行，如何根据不同的策略来优化代码。这些会在之后的习题课进行详细一点的讲解。


