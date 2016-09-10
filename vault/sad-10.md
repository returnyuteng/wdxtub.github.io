title: 软件架构与设计 第 10 课 Visualizing Software Architectures
categories:
- Technique
date: 2016-02-04 09:51:51
tags:
- CMU
- 架构
- 设计
---

前面我们了解了基本的建模概念和方法，现在我们来看看如何把整个模型可视化。

<!-- more -->

---

模型和可视化很容易搞混，因为它们是紧密相关的。模型是抽象的信息——一系列设计选择。可视化给这些设计选择具体的形式，我们能够来描述不同的模型。下面是一个对比：

![](/images/14546024342899.jpg)

具体的评价标准，也其实和建模的很类似，包括：

+ Scope & Purpose
+ Basic Type
+ Depiction
+ Interaction
+ Fidelity
+ Consistency
+ Comprehensibility
+ Dynamism
+ View Coordination
+ Aesthetics
+ Extensibility

具体的方式很多，例如

+ Text Visualizations
    + 优点：提供了一种通用的描述标记，很多编辑器可以用
    + 缺点：模型变大之后复杂度增加，没有办法描述图结构和复杂的交互

![](/images/14546029716427.jpg)

+ General Graphical Visualizations
    + 优点：可以创建好看的描述，没有信息隐藏
    + 缺点：没有深层语义，难以和其他的可视化图形连接

![](/images/14546030577559.jpg)

+ UML Visualizations
    + 优点：许多工具可以用，类似图形化描述，但可以添加  UML 语义
    + 缺点：没有一个通用的标准，难以描述什么时候开始什么时候结束，许多工具只支持标准 UML

![](/images/14546031639442.jpg)

那么如何创建新的可视化描述呢，可以遵循以下几个步骤：

1. Borrow elements from similar visualizations
2. Be consistent among visualizations
3. Give meaning to each visual aspect of elements
4. Document the meaning of visualizations
5. Balance traditional and innovative interfaces

