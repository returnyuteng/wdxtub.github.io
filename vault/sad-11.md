title: 软件架构与设计 第 11 课 Service-Oriented Architecture
categories:
- Technique
toc: true
date: 2016-02-04 09:51:56
tags:
- CMU
- 架构
- 设计
---

这一课是根据老师的课程安排添加的，之后应该会按照课本的脉络结合老师的思路来进行讲解。

<!-- more -->

---

## 什么是面向服务架构

简单来理解，服务计算就是利用现有的服务，通过逻辑组合和分支，来构造新的应用。这里的服务，是一个可以重用的组件，由服务提供者提供并由服务请求者请求。

课堂上介绍了很多概念和定义，但是我觉得其实意义不算特别大，理解 SOA 还是要抓住重点，重点其实很简单，一个是强调复用性，一个是强调拓展性。复用性值得是基本功能的模块化标准化，拓展性指的是用户量实际上已经由架构本身的特点进行了高层次的处理，很多是具体面对并发时候的技术细节。

最为突出的代表是 web 服务了，比方说网站提供一系列 api，我们不需要知道他们是怎么实现的，只要按照一定的规范去使用即可。仔细思考下前面的不同角色和使用场景，就可以得出下面的架构：

![](/images/14548907014827.jpg)

这里了解两个术语：WSDL(Web Services Description Language) 和 RESTful API(Represnetational State Transfer)，后者的著名代表就是 HTTP Request/Response。服务请求者通过 SOAP(Simple Object Access Protocol) 来和服务提供者沟通。

沟通就两个字，但是在 web 这里，不知道为什么，复杂度就蹭蹭往上飙，各种协议真是乱花渐欲迷人眼，可以看看下图：

![](/images/14548913482297.jpg)

这么多不同的子系统组合在一起，实际上就是一个 SOA 的架构，通过松散耦合来构造一个灵活、可拓展的系统。

不过说到底，SOA 可以说是一种设计思想，既然是思想，就可以应用在不同的层级，比方说

+ 在编程层级，可以用 SOA 的思想来进行组件设计——称之为 SCA(Service Component Architecture)
+ 在中间件环节，可以利用 SOA 来指导设计，比方说 ESB(single Enterprise Service Bus)
+ 在过程层级，SOA 可以用于集成和管理，甚至可以驱动架构设计
+ 在企业层级，可以通过 SOA 的思想具体把任务细分

具体到代码上，其实也还是老生常谈的几个概念，无非是重用性，扩展性，合作性，平台无关，语言无关这之类的。用水做例子，以前挖井打水，那么每个不同的部件就千奇百怪了，不同的桶不同的绳子不同的口径，现在直接水成为一种服务，打开水龙头就有，和你在哪里是谁无关，这就是 SOA 的思想。

也就是因为这种设计思路和不同的抽象层级，就有了 IaaS, PaaS 和 SaaS，这里主要说 Saas(Software as a Service)。举几个例子

+ 固话 - 多租户
+ 分时系统 - 多实例
+ 在线支付 - 多租户
+ 电子邮件 - 多租户

但是这两天 LinkedIn 和 Tabluea 被腰斩的股价说明，发展才是硬道理，SaaS 什么的，也就是炒炒概念罢了。

对于 SaaS 来说，提供定制化服务，实际上就是提供不同的配置方案，也就是说，一个足够灵活的系统，通过不同的配置（默认或定制），来进行统一服务。

最近的一个同样非常火热的概念就是云计算了，我也在连载对应的课程，可以结合来看看，从思想和工程两个不同的角度，也是很有意思的。比方说下面的几个原则：

+ Principle 1: Integrated Ecosystem Management for Cloud
+ Principle 2: Virtualization of Cloud Infrastructure
+ Principle 3: Service-Orientation for Common Reusable Services
+ Principle 4: Extensible Provisioning and Subscrption for Cloud
+ Principle 5: Configurable Enablement for Cloud Offerings
+ Principle 6: Unified Information Representation and Exchange Framework
+ Principle 7: Cloud Quality and Governance

合起来就是

![](/images/14548946708473.jpg)
（一定要用心感受下）

## SOA Reference Architecture (SOA-RA)

SOA-RA 把一个基于 SOA 的系统分隔成可重用的架构组件。下面是一个 SOA 的抽象分层

![](/images/14551174098822.jpg)

其他的例子：

![](/images/14551174702617.jpg)

![](/images/14551174911905.jpg)

![](/images/14551175074227.jpg)

一个更加清晰的描述：

![](/images/14551175363986.jpg)

加上不同的组件之后就是

![](/images/14551176317606.jpg)

这样分层的一个好处就是可以极大减少接口的数量，如下图所示：

![](/images/14551177297106.jpg)

接下来我们来说说 REST 设计，举个例子，某公司部署了一个 web 服务，可以让用户：

+ 获取部件的列表
+ 获取某个部件的详细信息
+ 提交一个购买请求（Purchase Order, PO）

那么用 REST 的方式来实现就是：

![](/images/14551179020138.jpg)

这样的实现有什么特点呢：

+ 客户端和服务器之间是 pull-based 的交互风格，也就是客户端在需要的时候，向服务器端请求数据
+ Stateless，从客户端发出的每个请求都包含所需的所有信息，不利用任何存储在服务器端的上下文信息
+ 可以通过缓存来提高网络的效率
+ 统一的接口（比如 HTTP GET, POST, PUT, DELETE）
+ 资源都是命名的（比如使用 URL 命名）
+ 不同的资源可能相互连接，允许从一个过程跳转到另一个过程（URL 的链接跳转）

综上所述，我们可以得到 REST web 服务设计的原则：

1. 找出所有需要通过服务暴露的概念实体（在上个例子中就是 part list, detailed part data 和 purchase order）
2. 给每个资源创建一个 URL
3. 把客户端只读的资源（使用 HTTP GET），和客户端可以修改的资源（使用 HTTP POST, PUT, DELETE）分类
4. 所有通过 HTTP GET 访问的资源都不应该有副作用，不能修改。
5. 在资源间增加超链接使得客户端可以在不同信息间跳转
6. 逐渐显示所有的信息，不要在一个响应中展示所有信息
7. 确定展示数据的 schema（如 DTD, W3C Schema, RelaxNG, Schematron），对于需要 POST 或者 PUT 的内容，也要提供响应的描述
8. 用 WSDL 文档来描述如何调用这些服务

整个的生命周期是

![](/images/14551192696261.jpg)

具体每个阶段及简要介绍

+ 建模：第一阶段，利用概念建模技术，自顶向下，用基于 WSDL 的分解方法，难点在于如何让方法调用包含更多的语义信息
+ 开发：包括设计、开发和测试，这一部分是通常的软件开发方法（Ration Unified Process，敏捷开发，瀑布模型），和基于 XML 的协议绑定（如 SOAP）
+ 发布：如何连接
+ 探索：从不同的 web 服务中找到最合适的

### 监控和管理

主要包括：

+ 性能监控：主要是保证其服务质量（Quality of Service, QoS）
+ 实现 Service Level Agreement(SLA)，是一个服务提供者和服务请求者间的协议，保证提供一定质量的服务
+ 异常处理
+ 访问控制
+ 数据和信息分析

### 重用

重用可以看作是一个循序渐进，甚至有点后期的工作，一个可能的发展阶段是：

![](/images/14551199022753.jpg)

可以看到随着业务的增长，重用的重要性才慢慢凸现出来。具体的方法有：

+ 利用面向对象思想重新设计
+ 重新设计架构
+ 直接推倒重来

## 云计算

云计算的目标是在用户间共享资源，具体的方式是通过 provisioning（针对服务提供者）和 subscription（服务消费者）。不同层级的资源共享就形成不同的云服务：

+ Infrastructure Cloud (e.g. Storage as a service, hardware and IT infrastructure management as a service)
+ Software Cloud (e.g. SaaS focusing on middleware as a service or traditional CRM as a service)
+ Application Cloud (e.g. Application as a Service, UML modeling tools as a service, social network as a service)
+ Business Cloud (e.g. business process as a service)

## Big Data as a Service

大数据的应用及处理的基本流程是：

![](/images/14551227915870.jpg)

+ Discover: 主要指大数据的发现及数据清洗
    + Data collection
    + Stakeholder communications
    + Documentation
    + Reverse engineering
    + Blind spots
+ Evaluate: 分析和评估这些数据
    + Diagnosis
    + maturity level
    + Adoption lifecycle view
    + Target state formulation
    + Strategization
+ Evolve: 根据这些数据来进行只能的进化，或者说对于某一特定领域的熟悉
    + Blueprint
    + Metrics and criteria
    + Technology option screening
    + Prioritization
    + Tradeoffs
+ Recommend: 利用新的信息进行信息或者资源的推荐
    + Roadmapping
    + Organization model
    + Resource and skillset
    + PMO governace
    + Action plan 

![](/images/14551236654152.jpg)

+ Date Gateway: ActiveMQ, Apache Camel
+ Data quality, ELT: OPEN Refine, talend\*
+ NoSQL Store: HBase, mongoDB
+ Analytics: HIVE, Impala
+ Reporting: pentaho, R
+ Visualization: visual.ly, Quantum4D
+ Machine Learning: WEKA, mahout
+ Cloud Hosting: Openstack, Amazon EC2
+ Monitoring Management: Ganglla, docker, hyperic

Best Practice

+ Iterative
+ Holistic
+ Pragmatic
+ Converged
+ Prescriptive
+ Incremental
+ Disciplined
+ Proactive

