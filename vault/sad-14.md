title: 软件架构与设计 第 14 课 Architecture Tradeoff Analysis Method
categories:
- Technique
toc: true
date: 2016-02-18 11:03:29
tags:
- CMU
- 架构
- 设计
---

这节课具体会介绍进行架构分析时候的一些权衡考量的方法，还是非常有用的（至少比前面两课那么『虚』的要好很多）。这里也会综合老师给出的阅读材料，争取给大家一个完整的概念。

<!-- more -->

---

## 阶段一 Evaluator & Decision Maker

### 第一步 Present ATAM

使用的技术有：

+ Utility tree genration
+ Architecture elicitation and analysis
+ Scenario brainstorming / mapping

这个阶段的产出有

+ Architectural approaches
+ Utility tree
+ Scenarios
+ Risks and "non-risks"
+ sensitivity points and tradeoffs

![ATAM 过程](/images/14558185071430.jpg)

### 第二步 Present Business Drivers

顾客代表会描述：

+ 系统的商业 context
+ 高层级的功能需求
+ 高层级的质量需求

> Architectural drivers: quality attributes that "shape" the architecture
> 
> Critical reirements: quality attributes most central to the system's success

具体来说包括：

+ 系统的关键功能
+ 任何技术、管理、经济或政策的限制
+ 商业目标和背景
+ 主要的权益关系着
+ 主要的质量目标(principla quality attribute, NFP)

### 第三步 Present the Architecture

展示系统的整体架构，可能包括：

+ 技术限制：操作系统、硬件、中间件
+ 其他需要与之交互的系统
+ 如何选择架构风格来达到质量要求

这个阶段评估小组开始寻找可能的风险

### 第四步 Identify Architectural Approaches

开始寻找那些在架构中堆达到质量目标举足轻重的关键点，了解比较常用的架构模型：

+ Client-Server
+ 3-tier
+ Proxy
+ Publish-Subscribe
+ Redundant hardware

### 第五步 Generate Utility Tree

通过构造 utility tree 来进一步优化改进重要的质量目标

+ Utility tree 是一个自顶向下的方法，来驱动属性相关的需求
+ 高层级节点应当是最重要的质量目标（如 performance, modifiability, security 和 availability）
+ 场景是 utility tree 的叶子

![一个 utility tree 的例子](/images/14558198859387.jpg)

**Scenarios**

用来展示利益相关者的兴趣所在，便于理解质量需求，应当包括

+ Anticipated uses of (use case scenarios)
+ Anticipated changes to (growth scenarios)
+ Unanticipated stresses (exploratory scenarios)

![ATAM Scenarios](/images/14558199929846.jpg)

![Scenario 的例子](/images/14558200171596.jpg)

### 第六步 Analyze Architectural Approaches

评估小组从特定的质量目标的角度入手，了解架构及识别可能的风险：

+ 找到那些保证质量需求的方法
+ 提出相关问题
+ 识别出 risks, non-risks, sensitivity points 和 tradeoffs

下面是一些例子：

![可能的问题](/images/14558202021169.jpg)

![Sensitivity & Tradeoffs](/images/14558202227194.jpg)

![Risk & Non-Risks](/images/14558202695271.jpg)

### 最后几步

+ 第七步 Brainstom & Prioritize Scenarios
    + 利益相关者头脑风暴，相关过程
+ 第八步 Analyze Architectural Approaches
    + 重复第六步中的工作
+ 第九步 Present ATAM results
    + 完整的产出
    + Architectural appoaches
    + Utility tree
    + Scenarios
    + Risks and Non-risks
    + Sensitivity points and tradeoffs

### 总结

+ 架构分析既不简单也不便宜
+ 从长远来看，利大于弊
+ 关于系统核心特性的早期信息非常重要
+ 应当使用多种分析技术
+ 到底需要分析到什么程度？
    + 这是架构师最需要琢磨的地方
    + 分析过多，则会无谓消耗有限的资源
    + 分析过少，则会面临系统失败的风险
    + 错误的分析既浪费资源，又可能导致系统失败

## 阅读材料笔记

+ 云计算部分非常简略，建议参考我的云计算课程笔记
+ 使用 ATAM 来评估一个架构时，目标是理解不同的架构决定对系统质量可能带来的影响，所以可以看作是一个风险识别的方法
+ 实际上是根据不同的属性和资源提出对应的问题，并找出相互的影响
+ 三个场景：use case scenarios, growth scenarios, exploratory scenarios
+ 不同的场景有不同的侧重点和要求，具体可以结合下图的实例来理解体会

![Use Case Scenarios](/images/14558122473154.jpg)

![Growth Scenarios](/images/14558122635076.jpg)

![Exploratory Scenarios](/images/14558122890374.jpg)

+ Utility tree 和 Facilitated brainstorm 是两种常见的方式，各有不同的侧重

![Utility Trees vs. Scenario Brainstorming](/images/14558123943034.jpg)

![一个 Utility Tree 的例子](/images/14558124405619.jpg)

+ ATAM 过程结束后的产出有
    + Risk and Non-Risks 文档，用来分析各个架构决定的风险性
    + Sensitivity and Tradeoff Points 文档，用来具体说明不同权衡的利弊（最重要的部分）
    + A Structure for Reasoning 文档，用来说明具体的架构选择及其原因

![Concept Interactions](/images/14558126249185.jpg)

+ 评测中的重要属性
    + Availability, Modifiability, Performance, Security, Usability
    + Deployment, Locating, Errors, Cost/Effort Estimation, Personalization, Safety
+ 可能的风险来源
    + Risks Due to Unknowns
    + Risks Due to Side Effects of Architectural Decisions
    + Risks Due to Ignoring Architectural Solutions to Attribute Requirements
    + Risks Due to Interaaction with Other Organizations
+ 具体步骤
    + Background
    + Business and Mission Drivers
    + Architectural Approaches
    + Utility Tree
    + Scenario Generation and Prioritization
    + Analysis Process
    + Post-ATAM Activities 

感觉还是有点虚，估计作业就是模仿阅读材料完成一次具体的 ATAM 分析，但是老师给出的参考作业的内容又是关于 SOA 的，真的很令人费解。

