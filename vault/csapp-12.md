title: 深入理解计算机系统 第 12 课 Linking
categories:
- Technique
toc: true
date: 2016-03-06 07:47:27
tags:
- 链接
- CMU
- 计算机
- 组成原理
---

这一课我们要接触一个新概念：Linking，简单来说就是计算机如何让不同的代码协同工作的方式。

<!-- more -->

---

我们先来看一个例子，假设有这么两个代码文件

![](/images/14572758203003.jpg)

我们用下面的命令来编译执行：

```bash
linux> gcc -Og -o prog main.c sum.c
linux> ./prog
```

编译器实际上会分别编译不同的源代码，生成 `.o` 文件，具体把这些文件链接在一起的是 Linker 链接器，整个过程如下图所示：

![](/images/14572760766987.jpg)

> 为什么要使用链接器？

有如下两个原因。

+ 模块化角度考虑。我们可以把程序分散到不同的小的源代码中，而不是一个巨大的类中。这样带来的好处是可以复用常见的功能/库，比方说 Math library, standard C library.
+ 效率角度考虑。改动代码时只需要重新编译改动的文件，其他不受影响。而常用的函数和功能可以封装成库，提供给程序进行调用（节省空间）

> 链接器做了什么？

主要负责做两件事情

**第一步：Symbol resolution**

我们在代码中会声明变量及函数，之后会调用变量及函数，所有的符号声明都会被保存在符号表(symbol table)中，而符号表会保存在由汇编器生成的 object 文件中（也就是 `.o` 文件）。符号表实际上是一个结构体数组，每一个元素包含名称、大小和符号的位置。

在 symbol resolution 阶段，链接器会给每个符号应用一个唯一的符号定义，用作寻找对应符号的标志。

**第二步：Relocation**

这一步所做的工作是把原先分开的代码和数据片段汇总成一个文件，会把原先在 `.o` 文件中的相对位置转换成在可执行程序的绝对位置，并且据此更新对应的引用符号（才能找到新的位置）

在具体来看这两步做了啥之前，先要理解下面几个概念。

## 三种 Object Files

所谓的 Object File 实际上是一个统称，具体来说有以下三种形式：

+ Relocatable object file (`.o` file)
    + 每个 `.o` 文件都是由对应的 `.c` 文件生成，包含代码和数据，可以用来组合成 executable object file
+ Executable object file (`a.out` file)
    + 包含代码和数据，可以直接被复制到内存中执行
+ Shared object file (`.so` file)
    + 在 windows 中被称为 Dynamic Link Libraries(DLLs)，是一类特别的 relocatable object file，能够被载入内存并动态链接（载入时或运行时）

## Executable and Linkable Format(ELF)

上面提到的三种 obejct file 有统一的格式，即 Executable and Linkable Format(ELF)，因为，我们把它们统称为 ELF binaries，具体的文件格式如下

![ELF 文件格式](/images/14572780724226.jpg)

下面分别介绍一下各个部分：

+ ELF header
    + 包含 word size, byte ordering, file type (.o, exec, .so), machine type, etc
+ Segment header table
    + 包含 page size, virtual addresses memory segments(sections), segment sizes
+ .text section
    + 代码部分
+ .rodata section
    + 只读数据部分，例如 jump tables
+ .data section
    + 初始化的全局变量
+ .bss section
    + 未初始化的全局变量
    + "Block Started by Symbol"
    + "Better Save Space"
    + 有 section header 但实际上不占空间
+ .symtab section
    + 包含 symbol table, procudure 和 static variable names 以及 section names 和 location
+ .rel.txt section
    + Relocation info for .text section
    + Addresses of instructions that will need to be modified in the executable
    + Instructions for modifying
+ .rel.data section
    + Relocation info for .data section
    + Addresses of pointer data that will need to be modified in the merged executable 
+ .debug section
    + 包含 symbolic debugging (`gcc -g`) 的信息 
+ Section header table
    + Offsets and sizes of each section

## Linker Symbols

而链接器实际上会处理三种不同的符号，对应于代码中不同写法的部分：

+ Global symbols
    + 在当前模块中定义，且可以被其他代码引用的符号，例如非静态 C 函数和非静态全局变量
+ External symbols
    + 同样是全局符号，但是是在其他模块（也就是其他的源代码）中定义的，但是可以在当前模块中引用
+ Local symbols
    + 在当前模块中定义，只能被当前模块引用的符号，例如静态函数和静态全局变量
    + 注意，Local linker symbol 并不是 local program variables

现在我们可以回过头来看看链接器具体做的工作了：

## 第一步：Symbol resolution

![Symbol resolution](/images/14572791281425.jpg)

我们可以看到，链接器只知道非静态的全局变量/函数，而对于局部变量一无所知。

然后我们来看看局部非静态变量和局部静态变量的区别

+ 局部非静态变量会保存在栈中
+ 局部静态变量会保存在 `.bss` 或 `.data` 中

例如：

![](/images/14572793537491.jpg)

如果两个不同的源代码中使用了相同的全局变量名称，链接器会如何处理呢？

首先我们需要知道的是，不同的符号是有强弱之分的：

+ Strong: procedures and initialized globals
+ Weak: uninitialized globals

比如：

![](/images/14572854731828.jpg)

在这个基础上，有如下规则：

![](/images/14572855304555.jpg)

因为这个特性，可能会因为变量重名导致非常奇怪的现象，比如下面的情况：

![](/images/14572855800167.jpg)

因此我们可以得到一条很重要的编程建议：

> 如果可能，尽量避免使用全局变量

如果一定要用的话，注意下面几点：

+ 使用静态变量
+ 定义全局变量的时候初始化
+ 注意使用 `extern` 关键字

## 第二步：Relocation

大概的过程，通过下图就可以看得比较清楚，就是把不同的 relocatable object files 拼成 executable object file 的过程

![](/images/14572863145911.jpg)

但具体是怎么做到的呢，还是刚才那个例子：

![](/images/14572863618645.jpg)

对应的 relocatable object file 反编译出来 `objdump -r -d main.o` 可以看到汇编代码为：

![](/images/14572864112503.jpg)

这里我们可以看到，编译器用 relocation entry 来标记不同的调用（注意看对应的代码后面四组数字都是零，就是留出位置让链接器在链接的时候填上对应的实际内存地址）

在完成链接之后我们得到 `prog` 这个程序，同样反编译 `objdump -dx prog` 可以看到：

![](/images/14572865303343.jpg)

对应的地址已经被填上去了，这里注意用的是相对的位置，比方说 0x4004de 中的 05 00 00 00 的意思实际上是说要在下一句的基础上加上 0x5，也就是 0x4004e8，即 sum 函数的开始位置。

具体载入内存的时候，大概是这样的：

![](/images/14572866787544.jpg)

这里需要注意左边的部分地址从上往下，右边则是从下往上，这里所有的程序都会从 0x400000 开始。

## 打包常用程序

基本上每个程序都会用到某些特定的函数，比如：数学计算, 输入输出, 内存管理, 字符串操作等等，我们能用什么方法把它们结合到程序中呢，有以下两个思路：

+ 思路 1：把所有的函数放到一个源文件中，程序员每次把这一整个大块头链接到自己的程序中，这种做法从时间和空间上来说都比较低效
+ 思路 2：不同的函数放到不同的源文件中，由程序员显式链接所需要的函数，这种做法效率更高，但是相当于是给程序员增加负担了

### Static Libraries

比较老式的做法就是所谓的静态库(Static Libraries, `.a` archive files)

+ Concatenate related relocatable object files into a single file with an index (called an archive)
+ Enhance linker so that it tries to resolve unresolved external references by looking for the symbols in one or more archives
+ If an archive member file resolves reference, link it into the executable

具体的过程如下：

![](/images/14572879938673.jpg)

这里注意，Archiver 支持增量更新，如果有函数变动，只需要重新编译改动的部分。

下面是两个常用的库：C standard library 与 C math library

![](/images/14572880658935.jpg)

接下来我们看一个具体的例子，通过静态库来连接：

![](/images/14572880967573.jpg)

具体过程如下：

![](/images/14572881242795.jpg)

具体的链接方式是：

![](/images/14572882030642.jpg)

但是这样会带来一个问题：写编译命令的时候，顺序是很重要的！我们看下面这个例子

![](/images/14572882337934.jpg)

第一条命令中，在编译链接的时候，如果在 libtest.o 中发现了外部引用，就会在 -lmine 中查找，但是如果反过来，在第二条语句中 libtest.o 后面没有东西，就会出现找不到引用的错误。所以建议就是要把静态库都放到后面去。

### Shared Libraries

现代的方法则是使用共享库，避免了在文件中静态库的大量重复。

动态链接可以在首次载入的时候执行(load-time linking)，这是 Linux 的标准做法，会由动态链接器 `ld-linux.so` 完成，Standard C library(libc.so) 通常是动态链接的。

![Dynamic Linking at Load-time](/images/14572885863404.jpg)


动态链接也可以在程序开始执行的时候完成(run-time linking)，在 Linux 中使用 `dlopen()` 接口来完成（会使用函数指针，如下面的例子所示），通常用于分布式软件，高性能服务器上。而且共享库也可以在多个进程间共享，这在后面学习到虚拟内存的时候会介绍。

![](/images/14572886125265.jpg)

![Dynamic Linking at Run-time](/images/14572886257737.jpg)

总结一下：

+ 链接使得我们可以用多个 object files 构造我们的程序
+ 链接可以发生在程序的不同阶段
    + 编译期间(when a program is compiled)
    + 载入期间(when a program is loaded into memory)
    + 运行期间(while program is executing)
+ 理解链接可以帮助我们避免遇到奇怪的错误

## Case Study: Library Interpositioning

这是一个非常有意思的技术，我们可以通过这个技术让程序运行任意我们想要的代码，比方说我们的程序中使用了 `malloc`，我们可以通过 library interpositioning 让程序执行我们自定义的 `malloc` 而不是标准库中的 `malloc`。

因为这相当于是某种链接技术，所以同样可以在不同的时候发生，如：

+ 编译时：When the source code is compiled
+ 链接时：When the relocatable object files are statically linked to form an executable object file
+ 载入/运行时：When an executable object file is loaded into memory, dynamically linked, and then executed.

这个技术可以应用在

+ 安全方面
    + Confinement (sandboxing)
    + Behind the scenes encryption
+ 调试方面
    + 可以找到隐藏比较深的 bug
+ 监控和查看性能
    + 统计函数调用的次数
    + 检测内存泄露
    + 生成地址记录

我们用一个具体的例子来说明，先来看看程序，非常简单，只有几行：

![](/images/14572895007610.jpg)

我们要做的事情也很简单，先申请一片内存空间，然后再释放掉。但是我们的目标是在不修改源代码的前提下，追踪分配地址的位置，要怎么办呢？

有三种方式，分别在编译、链接和运行时对 `malloc` 和 `free` 函数进行 interpositioning。

### Compile-time Interpositioning

我们写出另外两个函数，它们唯一做的事情就是输出地址，相当于把原来的函数做了个『包装』：

![](/images/14572896784067.jpg)

然后在 `malloc.h` 利用宏进行改变：

![](/images/14572897247442.jpg)

最后我们可以通过 `-I.` 这个选项来使得程序会调用我们自己写的函数，可以看到执行的时候会打印出地址

![](/images/14572897797778.jpg)

### Link-time Interpositioning

我们同样需要把两个函数包装一下：

![](/images/14572898402918.jpg)

然后注意所用的命令：

![](/images/14572898614846.jpg)

这里 `-Wl` 会告诉链接器，把每个逗号替换成空格。

`--wrap,malloc` 这个参数会进行特殊方式的引用

+ 对 `malloc` 的引用会被解析为 `__wrap_malloc`
+ 对 `__real_malloc` 的引用会被解析为 `malloc`

### Load/Run-time Interpositioning

我们同样是对两个函数进行包装：

![](/images/14572900345456.jpg)

![](/images/14572900442044.jpg)

然后使用以下命令：

![](/images/14572900675087.jpg)

这里的 `LD_PRELOAD` 环境变量会告诉动态链接器先在 `mymalloc.so` 中寻找所需的引用，就完成了 interpositioning 的效果。

