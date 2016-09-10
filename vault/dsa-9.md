title: 数据结构与算法 第 9 课 递归
categories:
- Technique
date: 2016-02-24 11:51:34
tags:
- CMU
- 数据结构
- 算法
---

递归之所以成为大家比较头疼的问题，主要还是其思维模式和我们惯常思考问题的方式不大一致，这里简要写下一些关于递归的碎碎念，希望能有所帮助。

<!-- more -->

---

我们一直在写程序，但是拿什么来证明程序本身的正确性呢？调试、测试用例、检查输出等等方法都没有办法保证百分百的正确性（黑天鹅效应）。

那怎么办呢？我们结合常见的编程模式来说明

+ Imperative Programming: Java 中常见的编程模式，大部分时间大部分人都是在按这种方式编程
+ Applicative Programming: 也就是比较出名的『函数式编程』，这里不具体展开，详情参阅[这里](https://zh.wikipedia.org/wiki/%E5%87%BD%E6%95%B8%E7%A8%8B%E5%BC%8F%E8%AA%9E%E8%A8%80)

其中函数式编程因为每个函数没有副作用，所以在概念上是比较好证明正确性的。接下来看看递归。

通用的模式是：

```java
recursive_fn(params){
    if (...) return some_value;
    else ... recursive_fn(new_params)..
}
```

另一种模式是尾递归 tail recursion

```java
tail_recursive_fn(params){
    if (...) return some_value;
    else return tail_recursive_fn(new_params)
}
```

通常来说，尾递归会更有效率一些，并且也更容易证明正确性。

另外一个比较常用的方法是回溯法，通用模式是：

+ Test if current position satisfies goal
+ If not, mark current position as visited and make a recursive call to search procedure on neighboring points
+ Exhaustive search, terminates as soon as goal is found

具体可以参阅我的『编程起跑线系列』

最后说一下递归的好处：

+ 实现起来比较简洁
+ 尾递归对栈空间也需求较少

## 总结

花了比较短的时间把这门课过了一次，总体来说不算特别全面，课件也是东拼西凑勉强合格，除了比较多的作业之外，感觉还是挺一般的。不过话说回来，除了名声在外的几门课，很多课程的教学质量也是堪忧的，唉。

