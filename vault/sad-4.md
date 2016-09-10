title: 软件架构与设计 第 4 课 Designing Architectures
categories:
- Technique
date: 2016-01-19 08:21:54
toc: true
tags:
- CMU
- 架构
- 设计
---

有了前面的基本概念和第一次作业的练习，这一讲我们主要来看看如何设计架构。

<!-- more -->

---

设计架构的目标主要有两个方面，一个是创新，一个是方法。创新的意思是拓展自己的技能树，使用一些新的工具；而方法的意思是去找到那些高效的技术。在开发的时候往往需要做一些取舍：什么时候用去开发新方法，什么时候去使用已经验证的有效方法。

具体到做工程，设计的过程主要分以下几个阶段：

+ Feasibility stage：可行性分析，找到一些可行的概念
+ Preliminary design stage：初步设计，选择和拓展最佳的概念
+ Detailed design stage：详细设计，做出这些概念的工程上的描述
+ Planning stage：计划阶段，根据产品，贡献，消耗和产品退出市场综合评估和改变这些概念

每一步实际上可能都不是那么一帆风顺，比如说：

+ 如果设计者没有办法产出可行的概念，那么整个流程就无法继续
+ 在问题越来越多，产品越来越大越来越复杂的时候，每一步成功的概率也在降低
+ 标准的方法没有充分去考虑市场因素，比如不同产品线之间的影响

所以主要矛盾就是日益增长的复杂度与设计者经验的不足之间的矛盾，为此，必须采用一些新的方法，比如：

+ Standard：也就是之前提到的线性模型
+ Cyclic：可以从后面的步骤跳回到之前的步骤
+ Parallel：独立的过程可以并行处理
+ Adaptive：就是走一步看一步
+ Incremental：每一个阶段可以看作是在上一个阶段的基础上进行优化

在继续讲下去之前，先说明若干概念，首先是『抽象』，抽象有两种方式，对应于软件工程中两类比较大的工程思路：自底向上和自顶向下。来看看下面的概念：

> Abstraction: The act or process of separating in thought, of considering a thing independently of its associations; or a substance independently of its attributes; or an attribute or quality independently of the substance to which it belongs.
> 
> Reification: The mental conversion of ...[an] abstract concept into a thing.
> 
> Deduction: The process of drawing a conclusion from a principle already known or assumed; spec. in Logic, inference by reasoning from generals to particulars; opposed to INDUCTION.
> 
> Induction: The process of inferring a general law or principle from the observation of particular instances; opposed to DEDUCTION.

最开始的时候，如果不知道从何入手，可以考虑 simple machine 方法，具体的意思就是，剥离大部分非核心的功能，从最基础的入手，比方说，要设计一个传真机，那么对应的 simple machine 就是一个状态机，先把这个状态机的状态确定好，然后再慢慢尽心迭代优化。对于不同的应用来说，通常都有对应的 simple machine，可以当作是万里长征第一步，一些常见的如下表所示：

Domain | Simple Machines
:--: | :--:
Graphics | Pixel arrays, Transformation matrices, Widgets, Abstract depiction graphs
Word processing | Structured documents, Layouts
Process control | Finite state machines
Income Tax Software | Hypertext, Spreadsheets, Form templates
Web pages | Hypertext, Composite documents
Scientific computing | Matrices, Mathematical functions
Banking | Spreadsheets, Databases, Transactions

如果决定使用抽象来作为设计工具，那么有两个问题必须要搞清楚，一个是抽象的层级，一个是需要讨论的范畴。这两个为什么重要呢？举个例子，假设我们在讨论宇宙，有意义的抽象的层级应该是一致的，不能说一个人在讨论行星恒星黑洞，另一个人在讨论水星火星地球。在层级一样之后，还需要确定的是具体到某个层级要讨论的范围，这样才能真正落到实处。

这里可以采用概念划分的方法，也就是把根据不同的子问题，划分出相互独立的几个部分，然后可以分别进行讨论。但是其实系统中各个部分或多或少都是相互联系的，那么如何分往往是需要取舍的。在软件工程中，一个比较关键的例子是组件（用于计算）和连接器（用于通信）直接的划分。

设计之中过去的经验也是很重要的，经验丰富的系统设计师经过长期训练所得到的直觉非常可靠（大部分情况下），我们还需要从过去的成功或失败中学习经验。失败是最好的老师，成功同样可以是，但是需要排除那些偶然因素影响的部分，去找到真正开启成功大门的钥匙。

然后我们来整体看一下不同部分在坐标系的位置，越靠近右上角，说明越需要领域相关的知识，同时也是从更高层的抽象来思考问题。

![](/images/14532127643541.jpg)

Domain-Specific Software Architecutres, DSSA 是一系列软件组件的组合，针对特定的领域，使用标准的组件。这种特性也使得 DSSA 只对特定的领域有关系，普适性较低。

Architectural Patterns 是一组架构设计决定，可以根据图中的位置来大概感受其特点。一个比较常见的三层模型为：

![](/images/14532130749730.jpg)

另一种是我们常常听到的 MVC，MVC 在移动应用中开发广泛使用（当然也有 MVVM）

![](/images/14532131812505.jpg)

而对于嵌入式带有各种传感器的应用，一般是这样的架构：

![](/images/14532132171147.jpg)

Architectural Styles 是软件系统设计的经验准则，需要更少的领域相关的内容。不同的风格之间是可以共存的。一些基本属性：

+ 设计元素的词汇表
	+ 包括组件类型，连接器类型，数据元素等等
	+ 例如：pipes, filters, objects, servers
+ 一组配置规则
	+ 不同组件间的限制与拓扑规则
	+ 例如：一个组件在当前条件下最多只能和其他两个组件相连
+ 语义描述
	+ 设计的组件需要有意义

使用 Styles 的好处有很多，例如：

+ 设计重用
+ 代码重用
+ 系统组织容易理解
+ Interoperability：也就是互用性
+ 特定风格分析
+ 可视化

一些可以问自己的问题：

+ What is the design vocabulary?
+ What are the allowable strucural patterns?
+ What is the underlying computational model?
+ What are the essential invariants of the style?
+ What are common examples of its use?
+ What are the (dis)advantages of using the style?
+ What are the style's specializations?

一些比较常见的风格（保证准确这里不翻译）：

+ Traditional, language-influenced styles
	+ Main program and subroutines
	+ Object-oriented
+ Layered
	+ Virtual machines
	+ Client-server
	+ MVC
+ Data-flow styles
	+ Batch sequential
	+ Pipe and filter
+ Shared memory
	+ Blackboard
	+ Rule based
	+ MapReduce
+ Interpreter
	+ Interpreter
	+ Mobile code
+ Implicit invocation
	+ Event-based
	+ Publish-subscribe
+ Peer-to-peer 
+ “Derived” styles 
	+ C2
	+ CORBA
	+ SOA

至此这一讲的内容就结束了，后面会详细说明不同风格。

