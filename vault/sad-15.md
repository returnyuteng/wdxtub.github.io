title: 软件架构与设计 第 15 课 Applied Architectures
categories:
- Technique
toc: true
date: 2016-02-26 16:06:41
tags:
- CMU
- 架构
- 设计
---

上节课我们了解了具体分析不同架构的方法，这节课我们主要来看看，如何应用不同的架构。

<!-- more -->

---

## 分布式/网络架构

类似『没有摩擦力』的小滑块，分布式架构是『乐观』建立在如下『完美』假设之上的，很多时候都难以满足：

+ 网络是可靠的
+ 延迟是没有的
+ 带宽是无限的
+ 网络是安全的
+ 拓扑结构是不变的
+ 只有一个管理员
+ 传输数据是不要钱的
+ 网络是多种多样的

虽然我们知道这些假设都是『美好的愿望』，不过我们还是借此建立起了一个庞大的分布式系统，也就是—— WWW（万维网）。作为一个分布式、去中心化的超媒体，整个 Web 的架构和代码是完全分离的，或者说，其实没有任何的代码去实现这个架构。我们有的，只是对于这个架构中不同组件的不同实现（比如说不同的浏览器）。

从架构的角度来看，就会发现互联网可能是这个世界上最成功的应用，更神器的是，居然没有任何代码去实现所谓『架构』！

所以我们知道，架构并不一定是一个有形的需要实现的东西，可能本身只是一种思想一种约定，不同的组件组合起来，就成为了架构。

### REST

我们总是能看到 RESTful API 这个词，但是到底是个什么意思？不妨从最基本的规则开始

1. 最关键的一步是把信息抽象成为以 URL 命名的资源。只要是能被命名的信息，就可以是一个资源
2. 资源的表示方法是一系列字节，加上自描述的 metadata，具体的表示形式可以由不同的 REST 组件协调确定
3. 所有的交互都是上下文无关的，每次交互都包含所需所有信息，不依赖于之前的请求
4. 每个组件只执行预定义好的方法，处理完成之后把资源传递给其他组件
5. 使用幂等操作和表达形式来支持缓存和重用
6. The presence of intermediaries is promoted. Filtering or redirection intermediaries may also use both the metadata and the representations within requests or responses to augment, restrict, or modify requests and responses in a manner that is transparent to both the user agent and the origin server.(不知道怎么翻译)

下面是一个例子：

![REST 的一个实例](/images/14565259010138.jpg)

在 REST 中的数据元素包含下面的内容：

+ Resource
+ Resource ID
+ Representation: Data + metadata
+ Representation metadata
+ Resource metadata
+ Control data

而具体的连接器有以下例子：

+ client: libwww, libwww-perl
+ server: libwww, Apache API, NASPI
+ cache: brower cache, Akamai cache network
+ resolver: bind (DNS lookup library)
+ tunnel: SOCKS, SSL after HTTP CONNECT

最后是组件的例子

+ User agent: brower
+ Origin server: Apache Server, Microsoft IIS
+ Proxy: Selected by client
+ Gateway: Squid, CGI, Reverse proxy(Controlled by server)

## 去中心化架构

所谓去中心化架构，实际上可以看成是某种意义上的『自治』与『分权』，网络中的不同组件可以有不同的行为，计算的过程也是分布式的。就好像我们现实生活中的合作一样。

比较有代表性了两个架构，就是我们下面要说的点对点和 web 服务。

### 点对点

去中心化的资源发现与共享，如 Napster 与 Gnutell，Skype 与 BitTorrent，也就是说没有一个所谓的『主服务器』来存储各种信息，每一台机子既可以看做是『客户端』，也可以认为是『服务端』。

+ 每个组件是相互独立的，有其自己的状态与控制线程。
+ 连接器：通常是自定义的网络协议
+ 数据元素：网络消息
+ 拓扑结构：动态随机有重复连接的网络

这种结构通过控制流和资源分布支持去中心化计算。即使有节点出现问题，也不会受到很大影响。

后面的内容比较常规（因为课本比较老 PS 老师年纪不大怎么也这么老派），大部分内容维基上的内容都比较详细了，这里不赘述，具体可以参考：

+ [Web服务](https://zh.wikipedia.org/wiki/Web%E6%9C%8D%E5%8A%A1)
+ [机器人学](https://zh.wikipedia.org/wiki/%E6%9C%BA%E5%99%A8%E4%BA%BA%E5%AD%A6)
+ [飞行模拟器](https://zh.wikipedia.org/wiki/%E9%A3%9B%E8%A1%8C%E6%A8%A1%E6%93%AC%E5%99%A8)

## 如何处理复杂性

虽然前面没有介绍飞行模拟器，但是因为需要比较现实的『模拟』，所以可以想到整个系统肯定是比较复杂的。于是我们怎么样能通过设计来减小复杂度呢？

+ 首先可以采用结构化建模的方法，基于面向对象设计来处理子系统和组件，目标是提高可维护性，集成性与可拓展性。
+ 软件部分，预先给不同的功能集合分簇，限制不合理的数据流与控制流，尽量减少数据类型。
+ 最后，每个组件保证封装性，计算不带副作用，不同组件之间要有交流和同步。

总结起来有以下四条

1. 好的架构是成功的一半
2. 好的架构源于对问题领域的深入理解
3. 好的架构可能是若干简单架构的结合
4. 开发新架构需要话很大力气，也要很小心。不过通常来说都不需要这么做。


