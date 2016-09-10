title: How to Write Fast Code 第 2 课 Multicore 编程
categories:
- Technique
toc: true
date: 2016-02-01 09:51:26
tags:
- CMU
- 编程
- 并行
- Multicore
- OpenMP
---

这一课主要是介绍 Multicore 编程以及 OpenMP 的相关内容，OpenMP 会另外专门做一个配套的教程，稍后共享给大家。

<!-- more -->

---

先要了解几个不同的并行层级，如下图所示：

![](/images/14543461995158.jpg)

OpenMP 实际上就是在代码中标记出可以并行执行的部分，由编译器来完成最终并行化处理的过程。具体的怎么做到的呢？参考下图：

![](/images/14543462820858.jpg)

OpenMP 这部分会专门写一次课来具体进行讲解，所以这里主要把理论上的东西提一下。

课程作业中另一个问题就是矩阵相乘，矩阵相乘是非常经典的问题，为了降低计算复杂度提高效率，这么多年来出现了各种各样的方法（不过基本也到极限了）。比方说利用分而治之的方式，利用内存排列的方式等等，具体来说可以考虑下面的方法:

+ Block size adaptation for appropriate caches
+ Register-level blocking
+ Copy optimization(data layout)
+ Optimizing the mini-matrix-multiply (base case)
+ Multi-level blocking
+ Multi-level copying

这部分也会专门写一课来进行讲解。

最后是很重要的一个用来分析性能的模型：Roofline Model。主要是理解下面几个图：

![](/images/14543502445260.jpg)

![](/images/14543502605039.jpg)

![](/images/14543502758805.jpg)

![](/images/14543502946889.jpg)

![](/images/14543503147538.jpg)

尤其是最后一张，超级重要：

![](/images/14543503391932.jpg)

具体的优化分类如下，大家在实践的时候可以思考下具体是在哪个类别进行优化，有没有其他类别的方法可以尝试：

![](/images/14543505552239.jpg)

最后再说一个概念：

![](/images/14543506076660.jpg)

可以通过汇编指令来进行查看

![](/images/14543506249554.jpg)


