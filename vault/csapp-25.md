title: 深入理解计算机系统 第 25 课 Thread-Level Parallelism
categories:
- Technique
date: 2016-04-08 09:52:18
tags:
- CMU
- 并行
- 线程
- 计算机
---

这节课我们从硬件讲起，然后介绍线程级并行，最后说一下一致性模型。这部分内容在我当助教的 18645 How to Write Fast Code 也有涉及。

<!-- more -->

---

回想一下，我们之前是如何处理 I/O 的延迟的呢？一个办法是每个客户端都由一个线程来处理，这样就不需要互相等待。现在的多核/超线程处理器提供了另外一种可能。我们不但可以并行执行多个线程，更好的是这些都是自动进行的，当然，我们也可以通过把大任务分成小任务来加速运算。

![Typical Multicore Processor](/images/14601449201982.jpg)

上图是典型的多核处理器架构，这里需要注意的是 L3 缓存和主内存都是共享的。而具体的执行计算的架构，基本的乱序执行处理器的架构为：

![Out-of-Order Processor Structure](/images/14601450728106.jpg)

指令控制器会动态把程序转换成操作流，操作会被映射到 Functional Unit 上进行并行处理。这种情况下，一个核心处理一个线程。而在超线程的设计中，一个核心可以处理若干个线程，秘诀在于多出来了若干套指令控制流，如下图：

![Hyperthreading Implementation](/images/14601452084625.jpg)

如果我们想要了解机器的相关信息，可以访问 `/proc/cpuinfo`

![](/images/14601453998378.jpg)

随后老师提及了两个例子，一个是并行求和，一个是并行快排，这里不赘述，如果感兴趣的话，可以自己思考一下。

另外一些关键字是

+ Amdahl's Law
+ Parallel Program Performance: Speedup & Efficiency

总结来看，并行编程需要注意的是：

+ 要有并行策略，可以把一个大任务分成若干个独立的子任务，或者用分而治之的方式来解决
+ 内循环最好不要有任何同步机制
+ 注意 Amdahl's Law

最后说一下内存一致性的问题，下面的程序会输出什么呢？

![Memory Consistency](/images/14601474171500.jpg)

具体输出的内容，取决于内存的一致性模型

![Sequential Consistency Example](/images/14601475352960.jpg)

如果使用 Write-back 的话，不同的缓存间不沟通，那么结果如下：

![](/images/14601476490089.jpg)

而更多的会用 tag 来标记不同的状态，由内存和缓存完成具体的协调工作：

![](/images/14601477220538.jpg)

![Before](/images/14601477302068.jpg)

![After](/images/14601477449024.jpg)

