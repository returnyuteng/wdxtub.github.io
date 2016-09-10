title: 深入理解计算机系统 第 16 课 Virtual Memory - Concepts
categories:
- Technique
toc: true
date: 2016-03-18 09:48:52
tags:
- 虚拟内存
- CMU
- 计算机
- 组成原理
---

这节课开始，我们会接触到计算机系统中另外一个非常重要的概念：虚拟内存。这个机制提供给了上层应用一个统一的地址空间，而无须操心物理内存的位置。

<!-- more -->

---

在具体讲述之前，我们先来看看什么是物理地址，什么是虚拟地址。

![物理地址](/images/14583091815694.jpg)

物理地址一般应用在简单的嵌入式微控制器中（汽车、电梯、电子相框等），因为应用的范围有严格的限制，不需要在内存管理中引入过多的复杂度。

但是对于计算机（以及其他智能设备）来说，虚拟地址则是必不可少的，通过 MMU(Memory management unit)把虚拟地址(Virtual Address, VA)转换为物理地址(Physical Address, PA)，再由此进行实际的数据传输。大致的过程如下图所示

![虚拟地址](/images/14583093850630.jpg)

使用虚拟内存主要是基于下面三个考虑：

1. 可以更有效率的使用内存：使用 DRAM 当做部分的虚拟地址空间的缓存
2. 简化内存管理：每个进程都有统一的线性地址空间
3. 隔离地址控件：进程之间不会相互影响；用户程序不能访问内核信息和代码

## 作为缓存工具

概念上来说，虚拟内存就是存储在磁盘上的 N 个连续字节的数组。这个数组的部分内容，会缓存在 DRAM 中，在 DRAM 中的每个 cache block 就称为 page（页），具体的大小为 $P=2^p$（这里 p 表示具体的位数），如下图所示：

![](/images/14583098368523.jpg)

大致的思路和之前的 cache memory 是类似的，就是利用 DRAM 比较快的特性，把最常用的数据换缓存起来。如果要访问磁盘的话，大约会比访问 DRAM 慢一万倍，所以我们的目标就是尽可能从 DRAM 中拿数据。为此，我们需要：

+ 更大的 page size：通常是 4KB，有的时候可以达到 4MB
+ 全相联 Fully associative：每一个 virual page 可以放在任意的 physical page 中，没有限制。
+ 映射函数非常复杂，所以没有办法用硬件实现
+ 通常使用 Write-back 而非 Write-through 机制
    + Write-through: 命中后更新缓存，同时写入到内存中
    + Write-back: 直到这个缓存需要被置换出去，才写入到内存中（需要额外的 dirty bit 来表示缓存中的数据是否和内存中相同，因为可能在其他的时候内存中对应地址的数据已经更新，那么重复写入就会导致原有数据丢失）

具体怎么做呢？通过 page table。每个 page table 实际上是一个数组，数组中的每个元素称为 page table entry(PTE)，每个 PTE 负责把 virtual page 映射到 physical page 上。在 DRAM 中，每个进程都有自己的 page table，具体如下

![Page Table](/images/14583104118576.jpg)

因为有一个表可以查询，就会遇到两种情况，一种是 Page Hit，另一种则是 Page Fault。

> Page hit: reference to VM word that is in physical memory (DRAM cache hit)

访问到 page table 中蓝色条目的地址时，因为在 DRAM 中有对应的数据，可以直接访问。

> Page fault: reference to VM word that is not in physical memory (DRAM cache miss)

访问到 page table 中灰色条目的时候，因为在 DRAM 中并没有对应的数据，所以需要执行一系列操作（从磁盘复制到 DRAM 中），具体为：

+ 触发 Page fault，也就是一个异常
+ Page fault handler 会选择 DRAM 中需要被置换的 page，并把数据从磁盘复制到 DRAM 中
+ 重新执行访问指令，这时候就会是 page hit

复制过程中的等待时间称为 demand paging。

仔细留意上面的 page table，会发现有一个条目是 null，也就是没有分配。具体的分配过程（比方说声明了一个大数组），就是让该条目指向虚拟内存（在磁盘上）的某个 page（但并不复制到 DRAM，只有当出现 page fault 的时候才需要赋值）

看起来『多此一举』，但是由于局部性原理，虚拟内存其实是非常高效的机制，这一部分最后提到了 working set 的概念，比较简单，这里不再赘述：

![working set](/images/14583113431902.jpg)

## 作为内存管理工具

前面提到，每个进程都有自己的虚拟地址空间，这样一来，对于进程来说，它们看到的就是简单的线性空间（但实际上在物理内存中可能是间隔、支离破碎的），具体的映射过程可以用下图表示：

![maping](/images/14583114512897.jpg)

在内存分配中没有太多限制，每个 virtual page 都可以被映射到任何的 physical page 上。这样也带来一个好处，如果两个进程间有共享的数据，那么直接指向同一个 physical page 即可（也就是上图 PP 6 的状况，只读数据）

虚拟内存带来的另一个好处就是可以简化链接和载入的结构（因为有了统一的抽象，不需要纠结细节），如下图所示：

![Simplifying Linking and Loading](/images/14583116383432.jpg)

## 作为内存保护工具

Page table 中的每个条目的高位部分是表示权限的位，MMU 可以通过检查这些位来进行权限控制（读、写、执行）

![Permission bits](/images/14583120873244.jpg)

## 地址翻译

开始之前先来了解以下概念：

![Summary of Address Translation Symbols](/images/14583133332593.jpg)

然后我们通过一个具体的例子来说明如何进行地址翻译

![Address Translation With a Page Table](/images/14583134329438.jpg)

具体的访问过程为：

+ 通过虚拟地址找到 page table 中对应的条目
+ 检查 valid bit，是否需要触发 page fault
+ 然后根据 page table 中的 physical page number 找到内存中的对应地址
+ 最后把 virtual page offset 和前面的实际地址拼起来，就是最终的物理地址了

这里又分两种情况：Page Hit 和 Page Fault，具体过程如下：

![Page Hit](/images/14583139236125.jpg)

1. Processor sends virtual address to MMU
2. MMU fetches PTE from page table in memory
3. MMU fetches PTE from page table in memory
4. MMU sends physical address to cache/memory
5. Cache/memory sends data word to processor

![Page Fault](/images/14583140218329.jpg)

1. Processor sends virtual address to MMU
2. MMU fetches PTEA from page table in memory
3. MMU fetches PTE from page table in memory
4. Valid bit is zero, so MMU triggers page fault exception
5. Handler identifies victim (and, if dirty, pages it out to disk)
6. Handler pages in new page and updates PTE in memory
7. Handler returns to original process, restarting faulting instruction

把这个和我们前面提到的 cache memory 结合起来就是：

![Integrating VM and Cache](/images/14583153762095.jpg)

+ VA: virtual address, PA: physical address
+ PTE: page table entry, PTEA = PTE address

但是我们会发现，这样其实还不够快，L1 cache 虽然快，为什么不能直接在 MMU 进行一部分的工作呢？于是就有了另外一个设计：Translation Lookaside Buffer(TLB)

+ Small set-associative hardware cache in MMU
+ Maps virtual page numbers to physical page numbers
+ Contains complete page table entries for small number of pages

我们使用 Virtual Page Number 部分当做访问 TLB 的索引，具体如下（和 cache memory 非常相似）：

![Accessing the TLB](/images/14583156462414.jpg)

同样分两个情况：TLB Hit 和 TLB Miss

> A TLB hit eliminates a memory access

![TLB Hit](/images/14583156886189.jpg)

> A TLB miss incurs an additional memory access(the PTE)

![TLB Miss](/images/14583157195012.jpg)

### Multi-Level Page Tables

Page table 的另一个问题就是，因为往往虚拟地址的位数比物理内存的位数要大得多，所以保存 page table entry(PTE) 也是一个问题。举个例子：

假设一个 page 的大小是 4KB($2^12$)，每个地址有 48 位，一条 PTE 记录有 8 个字节，那么要全部保存下来，需要的大小是：

$$2^{48} \times 2^{-12} \times 2^3 = 2^{39} bytes$$

整整 512 GB!

所以解决办法就是，多层的 page table，第一层的 page table 中的条目指向其他的 page table，然后再去寻找具体的地址：

![A Two-Level Page Table Hierarchy](/images/14583164443493.jpg)

具体的翻译过程如下：

![Translating with a k-level Page Table](/images/14583165384715.jpg)

## 总结

+ Programmer's view of virtual memory
    + Each process has its own private linear address space
    + Cannot be corrupted by other processes
+ System view of virtual memory
    + Use memory efficiently by caching virtual memory pages
        + Efficient only because of locality 
    + Simplifies memory management and programming
    + Simplifies protection by providing a convenient interpositioning point to check permissions



