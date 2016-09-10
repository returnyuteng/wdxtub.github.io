title: 软件架构与设计 习题课 6 设计模式进阶练习
categories:
- Technique
date: 2016-03-30 07:11:03
tags:
- CMU
- 架构
- 设计
- 习题
---

接着上次的练习，这次给出具体场景，来进行设计模式的应用，咱们直接看习题。

<!-- more -->

---

> 简单来说就是不同等级的领导可以批不同价格的订单，超出了价格范围就需要提交给上一级审批，这个时候用什么设计模式呢？如果具体的限制因素不止价格一个呢？

顾名思义，直接用『责任链模式』。在这种模式中，通常每个接收者都包含对另一个接收者的引用。如果一个对象不能处理该请求，那么它会把相同的请求传给下一个接收者，依此类推。

代码如下：

```java
// DecisionMaker.java
public abstract class DecisionMaker{
    public static int director = 10000;
    public static int vp = 25000;
    public static int president = 100000
    
    protected int price;
    protected string title;
    // next element in the chain
    protected DecisionMaker nextDM;
    
    public void setNextDM(DecisionMaker dm){
        this.nextDM = dm;        
    }
    
    public void makeDecision(int price){
        if (this.price > 0 && price <= this.price){
            System.out.println(title + "can make the decision for price "  + price);
        } else if (nextDM != null){
            System.out.println("Handle to higher level decision maker");
        }
    }
}

// DirectorDM.java
public class DirectorDM extends DecisionMaker{
    public DirectorDM(int price){
        this.price = price;
    }
}

// VPDM.java
public class VPDM extends DecisionMaker{
    public VPDM(int price){
        this.price = price;
    }
}

// PresidentDM.java
public class PresidentDM extends DecisionMaker{
    public PresidentDM(int price){
        this.price = price;
    }
}

// ExecutiveMeetingDM
public class ExecutiveMeetingDM extends DecisionMaker {
    public ExecutiveMeetingDM(int price){
        this.price = price;
    }
}

// DMDemo.java
public class DMDemo {
    private static DecisionMaker getChainOfDMs() {
        DecisionMaker director = new DirectorDM(10000);
        DecisionMaker vp = new VPDM(25000);
        DecisionMaker president = new PresidentDM(100000);
        // -1 means infinitive
        DecisionMaker exemeet = new ExecutiveMeetingDM(-1);
        
        director.setNextDM(vp);
        vp.setNextDM(president);
        prsident.setNextDM(exemeet);
        
        return director;
    }
    
    public static void main(String[] args){
        DecisionMaker dmChain = getChainofDMs();
        
        dmChain.makeDecision(5000);
        dmChain.makeDecision(12000);
        dmChain.makeDecision(32000);
        dmChain.makeDecision(302000);
    }
}
```

> 机场一般由塔台统一协调控制，各个飞机之间不会有练习沟通，这里更适合用什么设计模式？

一般使用中介者模式，用来降低多个对象和类之间的通信复杂性。这种模式提供了一个中介类，该类通常处理不同类之间的通信，并支持松耦合，使代码易于维护。中介者模式属于行为型模式。

```java
// Tower.java
public class Tower {
    public static void handleRequest(Plane plane){
        System.out.println("[" + plane.toString() + "] Requested.");
    }
}

// Plane.java
public class Plane {
    private String company;
    private String code;
    
    public Plane(String c, String co){
        company = c;
        code = co;
    }
    
    public sendRequest(){
        Tower.handleRequest(this);
    }
    
    @Override
    public String toSring(){
        return company + "-" + code;
    }
}

// Airport.java
public class Airport {
    public static void main(String[] args) {
        Plane p1 = new Plane("AA", "1000");
        Plane p2 = new Plane("Delta", "2000");
        
        p1.sendRequest();
        p2.sendRequest();
    }
}
```

> CPU 的调度算法一般是 Round Robin，什么设计模式比较合适呢？画出类图并写出伪代码。

（注）这个模式我也不是很确定

前端控制器模式（Front Controller Pattern）是用来提供一个集中的请求处理机制，所有的请求都将由一个单一的处理程序处理。该处理程序可以做认证/授权/记录日志，或者跟踪请求，然后把请求传给相应的处理程序。

图和伪代码略

> 弄一个消息系统，可以广播给所有的订阅者，什么设计模式比较合适呢？画出类图并写出伪代码。

当对象间存在一对多关系时，则使用观察者模式（Observer Pattern）。比如，当一个对象被修改时，则会自动通知它的依赖对象。观察者模式属于行为型模式。

图和伪代码略

> 同一个社区的方法除了颜色基本一致，什么设计模式比较合适呢？画出类图并写出伪代码。

（注）这个模式我也不是很确定

享元模式（Flyweight Pattern）主要用于减少创建对象的数量，以减少内存占用和提高性能。这种类型的设计模式属于结构型模式，它提供了减少对象数量从而改善应用所需的对象结构的方式。

图和伪代码略

> 给出一个实际生活中使用状态模式的例子

编译器的词法解析器

> 给出一个实际生活中使用状态模式的例子

JSON 解析器

> 给出一个实际生活中使用命令模式的例子

各种 shell


## 参考链接

+ [责任链模式](http://www.runoob.com/design-pattern/chain-of-responsibility-pattern.html)
+ [中介者模式](http://www.runoob.com/design-pattern/mediator-pattern.html)
+ [前端控制器模式](http://www.runoob.com/design-pattern/front-controller-pattern.html)
+ [观察者模式](http://www.runoob.com/design-pattern/observer-pattern.html)
+ [享元模式](http://www.runoob.com/design-pattern/flyweight-pattern.html)
+ [状态模式](http://www.runoob.com/design-pattern/state-pattern.html)
+ [访问者模式](http://www.runoob.com/design-pattern/visitor-pattern.html)
+ [命令模式](http://www.runoob.com/design-pattern/command-pattern.html)

