title: 软件架构与设计 第 20 课 设计模式
categories:
- Technique
toc: true
date: 2016-03-17 13:33:55
tags:
- CMU
- 架构
- 设计
---

前面讲了这么多『玄学』概念，这一次终于回到软件开发的主战场——设计模式！

<!-- more -->

---

设计模式可以算得上是软件架构中最有门道的一部分，也是大家最熟悉却又最陌生的一部分。熟悉是因为任何软件工程的课一定会提到，陌生则是因为『纸上得来终觉浅』，往往在实际应用中，会出现很多问题。总体来说，设计模式可以看作是解决某种特定问题的成功经验的提炼，学习设计模式，等于是站在巨人的肩膀上，能看得更远。

因为自己对这个话题也很有兴趣，所以就不按照老师课堂的讲述来编排了（实话说我觉得她讲的非常一般），课堂上主要介绍了工厂模式、单例模式、建造者模式和原型模式（本文都会涉及）。

本文主要写自己对不同模式，以及模式之间的一些思考。

## 先唠叨两句

从前提到『设计模式』，总觉得是特别高大上的东西。随着代码越写越多，接触的事物越来越广泛，慢慢开始意识到这些所谓的『规则』，是保护，也是束缚。也开始怀疑过去不知道为什么就会去『相信』的东西，比如说本文的『设计模式』，以及与之密切相关的『面向对象』。

学术界的一大问题在于，有的时候为了凸现自己的不同，会强行『发明』一些东西。层出不穷的理论与技术，结合不同时代的主题，颇有种『一代补丁一代神』的循环感。最近特别火的 AlphaGO，最终不可避免会遇到 PS4 的尴尬——是在模拟地球，还是在模拟地球仪？

最近几年听到看到的各路流派给自己贴光环的故事已经太多，从 OO 到 SOA，从 Vim 到 Emac，从 Windows 到 Linux/Unix，例子不胜枚举。问题在于，它们都想用一个东西解决所有问题，但是真有这么个东西，早就解决了你想要解决这个解决问题的问题了（是不是被绕了？）。不能只有一种声音，不能只有一种思路，甚至可以从《一九八四》中『偷』来这么个概念——双重思想。

C++ STL 库的作者关于面向对象说过这么一段话（虽有断章取义之嫌，但应该还是能表达出他的理念）：

> I find OOP technically unsound. It attempts to decompose the world in terms of interfaces that vary on a single type. To deal with the real problems you need multisorted algebras – families of interfaces that span multiple types. I find OOP philosophically unsound. It claims that everything is an object. Even if it is true it is not very interesting – saying that everything is an object is saying nothing at all. I find OOP methodologically wrong. It starts with classes. It is as if mathematicians would start with axioms. You do not start with axioms – you start with proofs. Only when you have found a bunch of related proofs, can you come up with axioms. You end with axioms. The same thing is true in programming: you have to start with interesting algorithms. Only when you understand them well, can you come up with an interface that will let them work.

就拿 Java 来说，所有东西都是对象，这跟所有东西都不是对象又有什么区别呢？纵观《设计模式》一书所说的 23 种设计模式，说白了就是：

+ 多组合少继承
+ 面向接口而非实现
+ 高内聚低耦合

其实都是平时听得太多以至于不太在意的概念。或者很多时候我们被『想当然』封印住了思考，本能就往 OO 或者 SOA 的方向跳了。

然后说说 SOA，从大三刚开始接触的时候，我就感觉非常不科学。SOA 的想法很好，方向也很好，但是遇到的一个悖论就是，在业务复杂到足以体现 SOA 的优势的时候，往往 SOA 本身已经复杂得没人弄得懂了。因为大家都是提供一个服务给别人调用，那么调用背后发生的事情就是一团乱麻了，基本上一个功能完成后就只有两个命运：用一段时间，需要更新的时候直接重写。

这种时候不妨回过头来看看『传统』的数据驱动编程，就会发现只有一直抓住问题的本质，才不至于人为增加太多复杂度（所以这个学期的云计算课程人为浪费大家时间我真心感受到了助教的恶意）。

> Keep It Simple, Stupid!

## 常见模式

+ 工厂模式的本质，实际上是以一个统一的角度去理解所有的资源，具体根据特定的标志符来进行对应处理
+ 抽象工厂的本质，其实就是一组配置文件，搞得那么玄乎，真心还就是几个文本文件可以解决的问题
+ 原型模式的本质，Unix 中的 `fork` 可以说是完美体现，反正我先原样搞出来一个，剩下的自己继续处理
+ 单例模式的本质，更像是中央集权，可以通过穿透层级进行信息的快速传递，减少消耗
+ 适配器模式的本质，就是带面具，和病毒欺骗细胞完成匹配一个意思

这样列下去还有很多，术语之所以存在，是为大家提供一个概念上的平台，在平台之上，就可以放开手脚自由发挥，而不是被平台所限制。

> 人法地、地法天、天法道、道法自然

多去观察，多去思考，而不是借由所谓的『权威』代替自己的劳动，才是不断进步的动力。

## 总结

那些我们习以为常觉得『自然』的东西，其实才是最重要的『模式』。

（这真是一篇头重脚轻的日志，实在没意思赘述概念了，具体在后面的参考链接都有）

## 参考资料

+ [图说设计模式](http://design-patterns.readthedocs.org/zh_CN/latest/)
+ [Java之美从菜鸟到高手演变之设计模式](http://blog.csdn.net/zhangerqing/article/details/8194653)
+ [设计模式](http://www.runoob.com/design-pattern/design-pattern-tutorial.html)
+ [从面向对象的设计模式看软件设计](http://coolshell.cn/articles/8961.html)


