title: 软件架构与设计 期中考试复习
categories:
- Technique
date: 2016-02-27 22:10:51
tags:
---

这一部分是根据老师给出的复习提纲总结的部分相关内容，但是由于架构这个东西太抽象，所以只能尽力去理解一下了。

<!-- more -->

---

一想到要背这么多东西我就头疼，尽量利用已有知识理解之后串起来。

> Why software architecture is important?

自己的一些思考，之所以重要是因为：

+ 软件架构影响到传统软件开发的每一个环节
+ 清晰的软件架构是项目正常推进的指引
+ 方便重用与修改
+ 易于分析和测试

> Various view points

这里应该就是[习题课 3 从不同视角描述系统](http://wdxtub.com/2016/01/31/sad-r3/)的内容，几个视角是：

+ Function View: 各个模块图，包括接口和依赖
+ Information View: 展示 schema，数据模型和数据流，用状态机图来描述系统的信息模型
+ Concurrency View: 软件的组件需要在哪里部署和运行
+ Deployment View: 主要的软件包是什么
+ Development View: 在不同地方运行时，系统是如何运行的。你会如何保证数据一致性？

> What does it mean by architecture through software lifecycle?

这个题目在问什么我都有点迷糊，是不是说软件架构在软件的生命周期中的作用？还是其他什么的，大概分下面几个阶段：

+ Requirements
+ Design 
+ Coding
+ Testing

不同的过程都会由架构来指引并且对架构的设计和演化有影响

> Will architecture design allow creativity

这个是开放性问题，我觉得需要回答的要点是：

+ 允许创新
+ 但是更要尊重前人的经验
+ 大部分情况下不需要全新的架构，需要在已有的基础上提高改进

> Canonical elements of software architecture

这题就是概念题了，不过之前接触了这么多次，应该有一些印象，主要最后的 configuration 可能比较容易忘

+ Component: computation
+ Connector: communication / coordination
+ Configuration: topology and constraints

> Architectural styles vs Architectural patterns

这是我一直比较迷糊的概念，会着重解释一下。

+ Architectural styles
    + Codify key constraints and architectural elements (components, connectors, configurations) found effective used in a family of software systems over a given time period
    + Client/Server, P2P, Object Oriented, Layered, Data-Flow, Pipe and Filter, Blackboard, Rule Based
+ Architectural patterns
    + A set of architectural design decisions that are applicable to a recurring design problem, and parameterized to account for different software development contexts in which that problem appears.
    + 3-tier (Stage-Logic-Display), MVC, Sense Compute Control

Compared to styles, architectural patterns are at a coarser level of granularity (design decisions versus actual architectural elements) and are inherently more domain specific

简单来说就是 style 会有更多的细节，涉及到具体的数据、对象、管道、客户端、服务器端什么的。pattern 的话就比较虚一点，具体的界限感觉也没有多么清晰，自圆其说即可

> Methods of evaluating design methods like styles and patterns

+ Vocabulary,
+ structural patterns
+ computational model
+ invariants
+ common examples
+ disadvantages
+ specializations

我估计记不住，总体来说就是从概念，定义，模型，优势，劣势等说明，注意逻辑。

> Methods of design

Greenfield design (entirely fresh start, no baggage)

+ Analogy searching
+ Brainstorming
+ Literature searching
+ Morphological Charts
+ Removing Mental Blocks
+ Insight from requirements/implementation

按照自己写作业的思路来回答即可

> Software Connectors

4 main roles: Communication, Conversion, Facilitation, Coordination

8 main types: Event, Stream, Procedure Call, Arbitrator, Data Access, Distributor, Linkage, Adaptor

这个我真心背不下来，随缘

> Benefits of first-class connectors

+ Software evolution
+ separation of concerns
+ modularity
+ pluggability

显式连接器其实是解耦的好方法，带来的好处自然就是更灵活，但是可能会有性能影响

> Architectural Modeling

An architectural model is an artifact that captures some or all of the design decisions that comprise a system’s architectureArchitectural modeling is the reification and documentation of those design decisions

Important things to “get right”: Consistency, Accuracy/Precision, Ambiguity

How to choose what to model? Cost/Benefit Decision

What do we model? Structure (Architectural element), static/dynamic behaviors, functional/non-functional aspects, Views/Viewpoints

How do we evaluate modeling techniques? Scope/Purpose, Basic Elements, Style, Static/Dynamic Aspects, Dynamic Modeling, Non-functional aspects, ambiguity, accuracy/precision, viewpoints, view consistencyModeling approaches: Generic, early ADLs, style-specific and domain-specific languages, extensible ADLs

> Service Oriented Architecture

+ Triangular Operational Model
+ SOA Standard Stack
+ SOA on Bilateral View
+ SOA Solution Lifecycle
+ Enterprise Service Bus
+ SOA Reference Architecture –  Software as a Service

> ATAM

+ Nine steps
+ Review what you do in the group homework
+ Link architectural styles with utility tree, and analyze risks and provide justifications

> People, Roles and Teams

Desired skills

+ Software development expertise
+ Domain expertise
+ Communicator
+ Strategist
+ Consultant
+ Leader
+ Technologist
+ Cost estimator
+ Cheerleader
+ Politician
+ Salesperson

