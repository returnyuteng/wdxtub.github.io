title: 数据结构与算法 第 3 课 链表、栈和队列
categories:
- Technique
toc: true
date: 2016-02-24 10:58:50
tags:
- CMU
- 数据结构
- 算法
---

链表、栈和队列是非常基本的数据结构，万丈高楼平地起，即使简单，也要认真对待。

<!-- more -->

---

## 链表

链表是非常基础和常用的一个数据结构，尤其是在解析器(parser)、游戏和搜索算法中。而且很多时候会用来实现下面的 ADT(Abstract Data Type)：

+ 堆栈 Stack
+ 集合 Set
+ 哈希表 Hash Table

我们来重温一下实现某个数据结构要做的五个事情：

1. Understand it Abstractly
2. Write a Specification
3. Write Applications
4. Select, Design, Implement
5. Analyze the Implementation

接下来我们以链表为例子，具体来过一遍前三个步骤，后面两个步骤属于实现的细节和实现之后的分析，这里暂时不涉及。

### 1.Understand it Abstractly

因为链表比较简单，所以一幅图就可以解释清楚：

![黄色部分是数据，绿色部分是指针](/images/14563356077459.jpg)

### 2.Specification

明确的规格说明包括：

+ 构造器，公有方法和属性的说明
+ 每个方法包含前条件/后条件
+ 说明应该与实现无关

这里用一个 `IntNode` 为例子

```java
public class IntNode {
    private int data;     // data
    private IntNode link; // pointer
    
    // 构造器
    public IntNode(int initialData, IntNode initialLink){
        data = initialData;
        link = initialLink;
    }
    
    // getter
    public int getData(){
        return data;
    }
    public IntNode getLink(){
        return link;
    }
    
    // setter
    public void setData(int newdata){
        data = newdata;
    }
    public void setLink(IntNode newlink){
        link = newlink;
    }
}


```

### 3.Application

根据前面的规格说明，具体来使用我们创建的数据结构。这里我们不直接把变量声明为公有，而是通过 getter 和 setter 来进行访问，等于是把变量名称和这个变量本身进行了解耦，即使改变了变量名，只要函数接口不变，对外的行为仍旧是不变的

### 链表小结

需要掌握的链表操作：

+ 插入
+ 删除
+ 检查是否有环
+ 保证程序的健壮性（主要是头为空的时候）

其他比较特别的操作：

+ 合并 N 个链表
+ 反转链表
+ 截取链表的一部分
+ 寻找链表的 1/N

## 栈与队列

栈和队列是比较典型的 ADT，所谓 ADT，就是实际上内存中没有类似的数据结构对应，具体的操作是人为增加的设定，是为 Abstract，但是同时它们也被当做数据类型来用，是为 Data Type，于是就成为 ADT。

因为比较简单的缘故，这里大概说一下要点：

### 栈

性质

+ 后入先出
+ Last-In / First-Out

支持的操作：

+ push - 入栈
+ peek - 查看栈顶
+ pop - 弹出栈顶元素

常见应用

+ 程序执行 - 函数调用和返回实际上就是入栈出栈的内容，详情见我的『深入理解计算机系统』系列
+ 解析 - Parsing
+ 计算 postfix 表达式的值 - 例如 `4 + 3` 可以写成 `4 3 +`

需要注意的问题

+ 在栈为空的时候执行 pop，会导致 underflow

实现方式

+ 数组实现 - 需要一个变量来标记栈顶位置
+ 链表实现 - 插入元素时对表头操作需要注意

常见题目

+ 括号匹配
+ 翻转字符串
+ 模拟递归（N 皇后问题）

### 队列

性质

+ 先入先出
+ First-In / First-Out

支持的操作：

+ Enqueue - 入队列
+ Dequeue - 出队列

常见应用

+ Round-robin 调度机制 - 处理器处理进程或服务器处理请求（负载均衡）
+ 输入/输出 处理
+ 网络中 packet 的排队处理

实现方式

+ 数组实现 - 需要一个变量来标记队列头及队列尾的位置
+ 链表实现 - 需要保存表尾，处理表头的时候注意操作顺序

与栈的组合

+ 利用一个栈和一个队列可以用来判断回文串

队列的进阶使用

+ 优先队列
    + 插入队列的元素有一定的顺序要求
    + 每次插入实际上是某种意义上搜索和排序的过程
    + 可以用数组来模拟实现
    + 可以看作是『最大堆』或『最小堆』


