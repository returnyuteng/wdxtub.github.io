title: 深入理解计算机系统 第 19 课 Dynamic Memory Allocation - Advanced Concept
categories:
- Technique
toc: true
date: 2016-03-19 17:32:34
tags:
- 内存分配
- CMU
- 计算机
- 组成原理
---

这节课我们来看看关于内存分配的延伸话题，包括更加复杂的选择机制以及垃圾回收等相关内容。

<!-- more -->

---

这节课主要介绍后面三种方法，都是用来记录未分配空间的，具体如下：

![Keeping Track of Free Blocks](/images/14584386982186.jpg)

## 显式 free 列表

主要的改动在于，只记录 free block，而不是所有的 block。因为是指针，所以不仅需要记录后一个也需要记录前一个；与此同时，仍然需要 boundary tag 来作为合并的辅助信息。具体的结构如下：

![Explicit Free Lists](/images/14584389197608.jpg)

因为是指针，逻辑上是连续的，但实际上可以是无序的，如下图：

![Logically vs Physically](/images/14584389533544.jpg)

分配空间的过程就是指针重新指向的过程：

![Allocating From Explicit Free Lists](/images/14584389995555.jpg)

然后我们来看看，当释放空间的时候，具体要把刚释放的 block，放在列表中的哪个位置呢？有两种策略：后入先出或按照地址排序。

![Freeing With Explicit Free Lists](/images/14584390706378.jpg)

接下来是 LIFO 策略的几个不同的情况，灰色表示已经分配的空间：

Case 1

![Case 1](/images/14584391317448.jpg)

Case 2

![Case 2](/images/14584391482644.jpg)

Case 3

![Case 3](/images/14584391607445.jpg)

Case 4

![Case 4](/images/14584391818093.jpg)

总结一下，与隐式列表相比

+ 因为只记录 free block，在内存几乎满的时候效率很高
+ 因为需要切分 block 以及维护列表，所以稍微复杂一点
+ 对于每个链接来说需要 2 个额外的 word 来记录前面一个 free block 和后面一个 free block

## Segregated free 列表

> Most common use of linked lists is in conjunction with segregated free lists. Keep multiple linked lists of different size classes, or possibly for different types of objects.

也就是说，每个不同大小 block 会在不同的列表里，对于较小的 size，一般会有单独的列表，对于稍大的 size，列表的范围也会更大，如下图所示：

![Each size class of blocks has its own free list](/images/14584404151349.jpg)

要分配一个大小为 n 的 block：

+ 搜索比 n 大的 free list 列表
+ 如果找到了合适的，切分 block 并且把剩余的放到对应的列表中（可选）
+ 如果没有合适的 block，找更大的 size
+ 重复上述过程，直到找到为止

如果确实找不到：

+ 向系统请求额外的堆内存（使用 `sbrk()`）
+ 在新的内存中分配对应的空间
+ 把剩余的空间放到最大 size 的列表中

释放空间时：

+ 合并 block 并放到对应的列表中

Seglist allocator 的优势：

+ 更高的吞吐量(log time for power-of-two size classes)
+ 更好的内存利用率
    + First-fit search of segregated free list approximates a best-fit search of entire heap
    + Extreme case: Giving each block its own size class is equivalent to best-fit

## 垃圾回收

所谓垃圾回收，就是我们不再需要显式释放所申请内存空间了，例如：

```c
void foo() {
    int *p = malloc(128);
    return; /* p block is now garbage*/
}
```

这种机制在许多动态语言中都有实现：Python, Ruby, Java, Perl, ML, Lisp, Mathematica。C 和 C++ 中也有类似的变种，但是需要注意的是，是不可能回收所有的垃圾的。

我们如何知道什么东西才是『垃圾』呢？简单！只要没有任何指针指向的地方，不管有没有用，因为都不可能被使用，当然可以直接清理掉啦。不过这其实是需要一些前提条件的：

+ 我们可以知道哪里是指针，哪里不是指针
+ 每个指针都指向 block 的开头
+ 指针不能被隐藏(by coercing them to an `int`, and then back again)

相关的算法如下：

![Classical GC Algorithms](/images/14584417083434.jpg)

这里我们主要讨论第一种 Mark-and-sweep collection 算法（居然已经有五十多年的历史了）

内存具体的分布，可以看做是一个有向图，每个 block 相当于一个节点，每个指针相当于一条边。那些不在堆中的且指向堆的指针称为根节点（如寄存器，栈，全局变量等），如下图所示：

![Memory as a Graph](/images/14584420668262.jpg)

### Mark and Sweep Collecting

这个机制可以在 malloc/free 的基础上实现。当空间不够的时候， 在每个 block 的头部增加一个额外的 mark bit。

+ Mark: Start at roots and set mark bit on each reachable block
+ Sweep: Scan all blocks and free blocks that are not marked

整个过程如下：

![Mark and Sweep Collecting](/images/14584422740964.jpg)

一个简易实现的思路：

![Assumptions For a Simple Iimplementation](/images/14584424812466.jpg)

代码如下：

![](/images/14584426898133.jpg)

在 C 语言中，我们可以使用 `is_ptr()` 来判断是否为指针，但我们也知道，C 指针可能指向 block 的中间，这个时候我们要如何找到 block 的开头呢？

![指向中间](/images/14584429046281.jpg)

 可以这么做

+ Can use a balanced binary tree to keep track of all allocated blocks (key is start-of-block)
+ Balanced-tree pointers can be stored in header (use two additional words)

![存储结构](/images/14584429821424.jpg)

## 内存相关陷阱

关于内存的使用需要注意避免以下问题：

+ Dereferencing bad pointers
+ Reading uninitialized memory
+ Overwriting memory
+ Referencing nonexistent variables
+ Freeing blocks multiple times
+ Referencing freed blocks
+ Failing to free blocks

具体看上述问题前，我们先来了解 C 语言中操作符的顺序以及优先级：

![C operators](/images/14584431989892.jpg)

下面是一些例子，一定要好好理解（指针什么的真的头疼）

![C Pointer Declarations](/images/14584433625063.jpg)

### Dereferencing Bad Pointers

这是非常常见的例子，没有引用对应的地址，少了 `&`

```c
int val;
...
scanf("%d", val);
```

### Reading Uninitialized Memory

不能假设堆中的数据会自动初始化为 0，下面的代码就会出现奇怪的问题

```c
/* return y = Ax */
int *matvec(int **A, int *x) {
    int *y = malloc(N * sizeof(int));
    int i, j;
    
    for (i = 0; i < N; i++)
        for (j = 0; j < N; j++)
            y[i] += A[i][j] * x[j];
    return y;
}
```

### Overwriting Memory

这里有挺多问题，第一种是分配了错误的大小，下面的例子中，一开始不能用 `sizeof(int)`，因为指针的长度不一定和 int 一样。

```c
int **p;
p = malloc(N * sizeof(int));

for (i = 0; i < N; i++) 
    p[i] = malloc(M * sizeof(int));
```

第二个问题是超出了分配的空间，下面代码的 for 循环中，因为使用了 `<=`，会写入到其他位置

```c
int **p;

p = malloc(N * sizeof (int *));

for (i = 0; i <= N; i++)
    p[i] = malloc(M * sizeof(int));
```

第三种是因为没有检查字符串的长度，超出部分就写到其他地方去了（经典的缓冲区溢出攻击也是利用相同的机制）

```c
char s[8];
int i;

gets(s); /* reads "123456789" from stdin */
```

第四种是没有正确理解指针的大小以及对应的操作，应该使用 `sizeof(int *)`

```c
int *search(int *p, int val) {
    while (*p && *p != null)
        p += sizeof(int);
    
    return p;
}
```

第五种是引用了指针，而不是其指向的对象，下面的例子中，`*size--` 一句因为 `--` 的优先级比较高，所以实际上是对指针进行了操作，正确的应该是 `(*size)--`

```c
int *BinheapDelete(int **binheap, int *size) {
    int *packet;
    packet = binheap[0];
    binheap[0] = binheap[*size - 1];
    *size--;
    Heapify(binheap, *size, 0);
    return (packet);
}
```

### Referencing Nonexistent Variables

下面的情况中，没有注意到局部变量会在函数返回的时候失效（所以对应的指针也会无效），这是传引用和返回引用需要注意的，传值的话则不用担心

```c
int *foo() {
    int val;
    
    return &val;
}
```

### Freeing Blocks Multiple Times

这个不用多说，不能重复搞两次

```c
x = malloc(N * sizeof(int));
//  <manipulate x>
free(x);

y = malloc(M * sizeof(int));
//  <manipulate y>
free(x);
```

### Referencing Freed Blocks

同样是很明显的错误，不要犯

```c
x = malloc(N * sizeof(int));
//  <manipulate x>
free(x);
//  ....

y = malloc(M * sizeof(int));
for (i = 0; i < M; i++)
    y[i] = x[i]++;
```

### Memory Leaks

用完没有释放，就是内存泄露啦

```c
foo() {
    int *x = malloc(N * sizeof(int));
    // ...
    return ;
}
```

或者只释放了数据结构的一部分：

![Freeing only part of a data structure](/images/14584447178117.jpg)

### 对策

我们可以使用下面的工具和方法来处理内存的 bug

![Dealing With Memory Bugs](/images/14584447590453.jpg)



