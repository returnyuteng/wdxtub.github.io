title: 软件架构与设计 第 6 课 Styles and Greenfield Design
categories:
- Technique
date: 2016-01-19 08:22:11
toc: true
tags:
- CMU
- 架构
- 设计
---

前一讲了解了简单的架构风格，这节课看一下稍微复杂一点的风格，以及针对 Greenfield 设计（类似于开放式问题）的相关内容。

<!-- more -->

---

其他的一些风格会稍微复杂一些，相当于是简单风格的组合，如：

+ REST
+ C2: Implicit invocation + Layering + other constraints
+ Distributed objects: OO + client-server network style, CORBA

## C2

An indirect invocation style in which independent components communicate exclusively through message routing connectors. Strict rules on connections between components and connectors induce layering.

组件包括独立，可潜在并行的信息产生器或消费者。连接器包括消息路由，可以过滤，翻译，转发通知或者请求。数据元素主要是消息和通知。

Layers of Componnets and connectors, with a defined "top" and "bottom", wherein notifications flow downwards and requests upwards.

![](/images/14532424727100.jpg)

## Distributed Objects: CORBA

用多种语言编写的对象在多个主机上运行。对象通过良好定义的接口提供服务。对象通过远程过程调用(Remote procedure calls, RPCs)在不同的主机，进程和语言边界间调用方法。

组件主要是对象，连接器是过程调用。数据元素是方法的参数，返回值和异常。

General graph of objects from callers to callees.

这种风格是 location, platform, language transparency 的，也就是说不受以上的限制。

![](/images/14532429040620.jpg)

## Observations

这是一种在不同架构中有不同表现的风格。主要关注问题（领域）和 resulting system。

下面是总结

![](/images/14532456127563.jpg)

![](/images/14532456260782.jpg)

![](/images/14532456351981.jpg)

![](/images/14532456453748.jpg)

## Design Recovery

主要的任务是检查已有的代码基和审查系统的组件，连接器和总体的拓扑结构。一个常见的方法是把实现层的实体通过聚类的方法组成架构元素，两种方法：句法聚类和语义聚类。

### Syntactic Clustering

主要是针对代码级实体的静态关系，不需要运转系统就可以操作，例如 coupling 和 cohesion。但是可能会漏掉一些细微的关系，毕竟是没有动态信息的。

### Semantic Clustering

语义包含句法，从定义上就能很清楚的得知这个方法力图包含系统领域知识的方方面面，这也导致了比较难以自动化进行。

## Greenfield

所谓 Greenfield 实际上就是说对于当前问题所处的领域大家都没有什么经验，有下面这三种指导思想：

+ Divergence: shake off inadequate prior approaches and discover or admit a variety of new ideas
+ Transformation: combination of analysis and selection
+ Convergence: selecting and further refining ideas

不断在基本的步骤里尝试直到找到比较可行的解决方案，我感觉来说就是不断试错找到最优解。下面列出一些基本方法：

+ 类比搜索：通过类比去找不相关但相似的问题，看看他们是怎么结局问题的，例如『神经网络』的提出
+ 头脑风暴：这个是老生常谈了，唯一需要注意的是不要因为某些不合理的规则限制了大家的想象力
+ 文献搜索：他山之石，可以攻玉
+ 词法表：先列出主要的功能，然后给出对应的解决办法，有了这些办法，再进行下一步的设计和优化
+ 跳出盒子：不要给自己设限，试试跳跃性思维

还可以尝试其他各种方法，但是注意一定要有所限制：

+ 不然会越来越复杂
+ 需要批判性思考和决定
+ 考虑研究和试错的花费
+ 隔离不确定的决定
+ 不断评估系统的需求，确定设计边界

最重要的，实在没有什么想法的话，多读读需求，俗话说得好，『读书百遍，其义自见』


