title: 软件架构与设计 习题课 5 设计模式练习
categories:
- Technique
toc: true
date: 2016-03-20 20:43:05
tags:
- CMU
- 架构
- 设计
- 习题
---

这次的作业就是对设计模式的练习和熟悉，直接来看情境吧。

<!-- more -->

---

> Does the following code fragment implement the Factory Method design pattern? Explain why.

```java
public class XMLReaderFactory {
    // This method returns an instance of a class
    // that implements the XMLReader interface.
    // The specific class it creates and returns is
    // based on a system property.
    public static XMLReader createXMLReader();
}

public interface XMLReader {
    public void setContentHandler(ContentHandler handler):
    public void parse(InputStream is);
}
```

这部分代码片段实现了工厂模式，注释中说明是根据系统属性来进行创建，所以无须传入参数，需要被生成的对象都需要实现 XMLReader 接口。

> Typically, there is a significant performance penalty for declaring a method synchronized. In general adding the keyword synchronized to a method slows the invocation of the method by a factor of 6. A common programming idiom used to avoid synchronization in the common case is double check locking. Can the double-check locking idiom be used with the singleton pattern to avoid synchronizing every time the static instance method is called? For example, is the following valid? Explain why.

```java
public class Singleton {
    private Singleton() { }
    static private Singleton instance = null;
    static public Singleton instance() {
        // Double check locking idiom
        // The synchronization is done internally 
        // rather than on the method. This avoids
        // the expense of synchronizing for the
        // common case.
/*12*/  if( instance == null ) {
            synchronized( Singleton.class ) {
                // Second check
                if( instance == null ) {
/*16*/               instance = new Singleton( );
                 }
            }
        }
/*20*/  return instance;
    }
}
```

在大部分情况下代码没有问题，不过为了保险起见，还是要加上 `volatile` 关键字，不然在下面的情况中就会出问题：

1.	线程A发现变量没有被初始化, 然后它获取锁并开始变量的初始化。
2.	由于某些编程语言的语义，编译器生成的代码允许在线程A执行完变量的初始化之前，更新变量并将其指向部分初始化的对象。
3.	线程B发现共享变量已经被初始化，并返回变量。由于线程B确信变量已被初始化，它没有获取锁。如果在A完成初始化之前共享变量对B可见（这是由于A没有完成初始化或者因为一些初始化的值还没有穿过B使用的内存(缓存一致性)），程序很可能会崩溃。

[这里](https://zh.wikipedia.org/wiki/%E5%8F%8C%E9%87%8D%E6%A3%80%E6%9F%A5%E9%94%81%E5%AE%9A%E6%A8%A1%E5%BC%8F)有一个很好的参考资料

> Consider the following scenario, please suggest a design pattern and implement it. The outcome should like this:

![](/images/14585221758261.jpg)

In Elizabeth's day care center, the teacher helps the kids to build all kinds of toys to develop their creative skills. One of Elizabeth's favorite activities is to make animals with play-dough.

A set of molds is the tool that Elizabeth always uses to create her favorite cool animals.

One mold tool set includes five parts, including the head, body, arm, leg, and tail. Whenever Elizabeth wants to build an animal, she will use one set of each tools to make a head, body , leg, arm, and tail for that animal, and then assembles them with glue to build an animal. There are many types of animal mold tool sets that the kids can choose from.

For example: if Elizabeth wants to make a monkey, then she will pick the set of monkey molds to start.

+ Step 1. Make monkey head.
+ Step 2. Make monkey body.
+ Step 3. Make monkey leg.
+ Step 4. Make monkey arm.
+ Step 5. Make monkey tail.

Once all the five parts are finished, then Elizabeth will glue them all together and decorate it to have a monkey done as a finished product. Most likely, she will give it to her mom as a gift when she picks her (she will not give her the monkey if the monkey is not decorated, since it will not be looking good at all then). When she wants to make a kitten, she follows the same steps with the set of Kitten molds.

这道题目非常简单粗暴，看描述就知道直接使用建造者模式即可，具体的介绍看[这里](http://www.runoob.com/design-pattern/builder-pattern.html)

实现的话这里就不贴代码了，反正是不难的。

> You are working on software that interacts with a new hardware device and are anxious to test against the actual hardware. The hardware manufacture has a beta version of the driver they plan to release but is warning the interface of the device driver could change between now and the release date. The device driver is used throughout your code and you are concerned about writing code to an interface that is subject to change. What design pattern can be used to mitigate the risks involved? Describe what the risks are and how the design pattern mitigates these risks.

这个其实也比较简单，直觉反应就是适配器模式，用来做不同接口的转换。

> In keeping with the holiday theme, use the Decorator design pattern to model a Christmas tree as a base component (the tree) and optional decorations or adornments such as bulbs, candy and garland. Initially it's enough for your solution to model just the printing or display of the tree along with its adornments. You can also assume that each type of decoration is added all at once.

![](/images/14585236303062.jpg)

Show the class diagram for your solution and the runtime organization of objects for different configurations of adornments.

这里题目已经指名说要用装饰器模式，就按照[这里](http://www.runoob.com/design-pattern/decorator-pattern.html)的方法设计一下即可，我就不啰嗦了。

