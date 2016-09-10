title: 软件架构与设计 第 8 课 Choosing Connectors
categories:
- Technique
date: 2016-01-19 08:24:49
toc: true
tags:
- CMU
- 架构
- 设计
---

上一讲中我们了解了连接器的基本概念，这一讲我们来看看，在这么多不同的选项中，如何找到最适合项目的连接器。

<!-- more -->

---

具体到连接器的选择，涉及到选择，我们就应该意识到，上一讲其实是了解了选项，这一讲来说说如何选择。但是实话说，不要说一讲能说清楚，一百讲估计都不行，因为对于现实生活来说，本来就没有十全十美的选择。

工程没有人生这么复杂，可是需要考虑的东西仍然太多，看得到的是几个选项，看不到的是选项背后错综复杂的关系网。就拿连接器来说，基本功能，如何交互，这是需要考虑的，交互下面分几个大方向：communication, coordination, conversion, facilitation，根据这些选择合适的类型，确定作用的范围，再根据实际需要进行组合。

一个描述交互的模型，叫做 Interconnection models(IM)，从微观到宏观有三个层次：unit, syntactic, semantic。接下来简要介绍一下这几个不同的层级

## Unit Interconnection

定义了不同系统单元（组件，模块合作文件）之间的关系，最基础的单元关系就是依赖。下面是一些例子：

![](/images/14532537450817.jpg)
是比较粗粒度的静态相互连接，并不能描述组件之间的交互，主要是集中于相关性的描述

## Syntactic Interconnection

描述句法元素间的关系，比方说变量定义和使用，函数的声明和调用。下面是一些例子：

![](/images/14532538558847.jpg)
是比较细粒度的相互连接，可以是动态也可以是静态的。但是实际上是不完整的相互连接的描述，因为可能在语义层级是不允许的。

## Semantic Interconnection

描述系统组件如何被使用，可以正规进行定义：

+ 前条件
+ 后条件
+ 动态交互协议（例如 CSP, FSM）

举个例子：

![](/images/14532542194874.jpg)

基于 syntactic interconnections，可以是静态或者动态的，在架构层面是必须的，包括大的组件，复杂的交互以及组件重用。当然还有其他很多东西要考虑，如：鲁棒性，可靠性，安全性，可用性等等。

## 组合基本的连接器

许多时候我们需要组合多个连接器，但是要注意，不是所有的都可以搭配，有些是会相互冲突的，并且各有各的妥协。它们的关系为：

![](/images/14532544518305.jpg)

其实是有一张很大的表格：

![](/images/14532545047453.jpg)

这里不展开，不过列出一些知名的复合连接器：

+ Grid Connectors(e.g. Globus)
	+ Procedure call
	+ Data access
	+ Stream
	+ Distributor
+ Peer-to-peer connectors (e.g. Bittorrent)
	+ Arbitrator
	+ Data access
	+ Stream
	+ Distributor
+ Client-server connectors
+ Event-based connectors

这一讲的内容就这么多，其实还是蛮抽象的，之后的习题课会用具体的例子来分析，现在大概感受一下即可。
 


