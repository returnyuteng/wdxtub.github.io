title: 数据结构与算法 第 2 课 基础知识
categories:
- Technique
toc: true
date: 2016-02-24 10:35:52
tags:
- CMU
- 数据结构
- 算法
---

这一课我们不会涉及具体的数据结构或算法，而是先了解一些分析数据结构和算法的基本知识，如前条件/后条件，时间空间复杂度分析。

<!-- more -->

---

## 前条件/后条件

建议在写具体的函数之前，都要先想好（写好）前条件和后条件，这样可以确定问题的边界。

+ 前条件：在函数调用之前一定为真的条件
+ 后条件：在函数执行完之后为真的条件

例如：

```java
// Precondition: x >= 0
// Postcondition: The square root of x has
// been written to the standard output
public void writeSqrt(double x){
...
}
```

## Big O

在具体介绍之前，先来一些基本的数学公式复习

{% raw %} $$log_b(x_1·x_2)=log_bx_1+log_bx_2$$ {% endraw %}

{% raw %} $$log_b(\frac{x_1}{x_2})=log_bx_1-log_bx_2$$ {% endraw %}

{% raw %} $$log_b(x^c)=c·log_bx$$ {% endraw %}

事实上除了我们常用的 Big O，还有另外两种表示的方法：Big Omega($Big- \Omega$) 和 Big Theta($Big - \theta$)

这里只介绍它们的差别，具体的定义可以自行查看。

Big-O 实际上只表示上限，比如说，我们知道 $17n^2\in O(n^2)$，但是同时我们也可以说 $17n^2\in O(n^37)$ 和 $17n^2\in O(2^n)$

Big omega 则是表示下限，例如 $f(n) = n$，那么下面两个式子都成立 $f(n) \in O(n^2)$ 与 $n^2 \in \Omega(n)$

总结一下：

![](/images/14563304975385.jpg)

而 Big theta(Big $\theta$) 则是前面两个的交集：

$$\theta(f)=O(f) \cap \Omega(f)$$

举例来说，函数 $f(n) = 4n$，则

$$f(n)\in O(n)$$

$$f(n) \in \Omega(n)$$

所以

$$f(n)\in \theta(n)$$

简单来说就是

1. Big O 是一个算法最坏情况的度量
2. Big Omega 是一个算法最好情况的度量
3. Big Theta 表达了一个算法的区间，给定了一个函数的渐近的逼近(asymptotically tight bound)

但是一般来说用 Big O 也就足够了

## 面向对象编程

> Class = Data + Methods

具体要说的话，内容就太多了，需要深入理解『继承』『多态』和『封装』，也不是一篇日志能说清楚的，所以就大概有一个印象，慢慢领悟吧。




