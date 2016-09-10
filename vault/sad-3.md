title: 软件架构与设计 第 3 课 Basic Concepts
categories:
- Technique
date: 2016-01-14 17:27:06
toc: true
tags:
- CMU
- 架构
- 设计
---

在总体介绍了软件架构之后，这一讲主要介绍基本的概念，为之后的深入夯实基础。其中很多定义不是特别好翻译，就直接引用英文原文了。

<!-- more -->

---

什么是软件架构：A software system's architecture is the set of **principal design decisions** about the system.

主要包括：

+ 结构
+ 行为
+ 交互
+ 非功能属性 NFP

什么叫做 **Principal**？

意味着这个设计决定的架构状态，说明并不是所有的设计决定都是架构级别的；具体的标准就有软件开发的目标决定。

从时间的角度来看，在每个给定的时间点，有且仅有一个架构，而架构本身也是在不断变化的。

## Prescriptive vs. Descriptive Architecture

这是一个比较重要的概念，所以给出英文原文：

+ A system's prescriptive architecture captures the design decisions made prior to the system's construction.
	+ It is the **as-conceived** or **as-intended** architecture
+ A system's descriptive architecture describes how the system has been built
	+ It is the **as-implemented** or **as-realized** architecture

对于这两个概念，可以这么理解，prescriptive 是指在没有开发之前进行的设计，如果用盖房子来做例子，就相当于设计图纸；而 decriptive 则是指在软件开发出来之后，我们用来描述的架构，等于是房子已经盖好了，我们通过观察得到的架构。

这两种架构很多时候是不一样的，毕竟从纸上的设计到最终的制品，会有各种变化。当系统进化的时候，理想状况下首先修改 prescriptive architecture，而在实际生活中，通常就直接堆系统，也就是 descriptive architecture 进行改动了。有很多原因，例如：

+ 开发者马虎偷懒
+ 截止日期临近没有办法详细思考并撰写文档
+ 缺少 prescriptive architecture 的文档
+ 需要进行代码优化
+ 技术和工具支持不足

## Architecural Degradation 架构退化

包含两个相关的概念：架构漂移 Architectural drift 和 架构腐烂 Architectural erosion.

架构漂移指的是在 descriptive architecutre 中引入的重要设计决定没有包含在 prescriptive architecture 中，但是没有违背其设计决定。

架构腐烂指的是在 descriptive architecutre 中引入的重要设计决定违背了 prescriptive architecture 中的设计决定。

因为这种情况的发生，所以我们需要架构恢复，也就是实现层面上检查软件系统的架构，找出违背设计决定的地方并进行修正

## Deployment 部署

就是把程序放到它应该运行并发挥作用的地方。需要考虑的问题有：

+ 可用的内存
+ 能源消耗
+ 需要的网络带宽

![sad8](/images/sad8.jpg)

一个软件系统的架构通常不是，也不应该是一个单一庞大的组织，而应该是不同元素的组合和互动。

## Components 组件

定义：

+ 封装了系统功能和/或数据的一个子集
+ 通过显式定义的接口来限制访问
+ 有明确定义的执行所需的上下文

组件通常提供应用相关的服务

## Connectors 连接器

定义：连接器是影响和规范不同组件间交互的架构模块。

在复杂的系统交互中，连接器的作用比组件本身还要大。在许多软件中连接器只是简单的过程调用或者共享数据访问，但更加复杂的连接器是存在的。

Connectors typically provide application-independent interaction facilities.

一些连接器的例子：

+ 过程调用连接器
+ 共享内存连接器
+ 消息传递连接器
+ 流连接器
+ 分布式连接器
+ 包装/适配连接器

## Configurations 配置

定义：An architecural configuration, or topology, is a set of specific associations between the components and connectors of a software system's architecture.

一个例子：

![sad9](/images/sad9.jpg)

## Architectural Styles 架构风格

定义：

An architectural style is a named collection of architectural design decisions that

+ are applicable in a given development context
+ constrain architecural design decisions that are specific to a particular system within that context
+ elicit beneficial qualities in each resulting system

## Architectural Patterns 架构模式

定义：

An architectural pattern is a set of architectural design decisions that are applicable to a recurring design problem, and parameterized to account for different software development contexts in which that problem appears.

在现代分布式系统中广泛应用三层(three-tiered)系统模式

![sad10](/images/sad10.jpg)

+ Front Tier: 包含用户界面并以此访问系统服务。
+ Middle Tier: 包含应用的主要功能
+ Back Tier: 包含应用的数据和存储功能

其他的一些概念：

+ Architecture Model: An artifact documenting some or all of the architectural design decisions about a system
+ Architecture Visualization: A way of depicting some or all of the architectural design decisions about a system to a stakeholder
+ Architecture View: a subset of related architectural design decisions

## Architectural Processes 架构过程

+ 架构设计
+ 架构建模与可视化
+ 架构驱动的系统设计
+ 架构驱动的系统实现
+ 架构驱动的系统部署，运行时重部署，以及灵活性
+ 针对非功能属性基于架构的设计，包括安全和信任
+ 架构适配

## Stakeholders in a System's Architecture

+ 架构师
+ 开发者
+ 测试者
+ 管理者
+ 消费者
+ 用户
+ 销售者

