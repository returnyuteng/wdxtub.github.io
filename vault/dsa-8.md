title: 数据结构与算法 第 8 课 有限状态机
categories:
- Technique
toc: true
date: 2016-02-24 11:49:42
tags:
- CMU
- 数据结构
- 算法
---

《模仿游戏》的大热让图灵为更多人所知，不过电影中更多说的是图灵破解密码，却对图灵的另一个伟大设想——图灵机所言甚少。图灵机可以看作是某种有限状态机，虽然这个名词听起来比较陌生，但也许这是计算机学科中最重要的概念之一。

<!-- more -->

---

最先知道有限状态机，是在本科学习编译原理的时候，实话说，这类概念其实是很难理解的，不过第一次看不懂也不要紧，指不定哪天就悟道了。

我们先来看看常见的集中语言类别及其对应的计算模型：

![](/images/14564350412980.jpg)

## 基础概念

我们从最基本的概念说起，为之后进入更加抽象的概念打一些基础。

$\Sigma$: 一个有限的符号集合称为字母表。在 $\Sigma$(字母表)中的每个符号我们称为字母，通常用小写表示，如 a, b, c, ...

一个单词 w 是一个字符串，字符串中的字符都来自于 $\Sigma$，$|w|$ 表示单词 w 的长度，一个空的字符串不包含任何字母，并且用 $\varepsilon$ 表示。

一门语言 L 是一个由 $\Sigma$ 中的单词组成的集合，给定字母表 $\Sigma$，所有可能的字符串记为 $\Sigma^*$。例如，如果 $\Sigma=\{a\}$，那么 $\Sigma^* = \{\varepsilon, a, aa, aaa, \dots\}$

给定字母表 $\Sigma$，所有可能的长度为 i 的字符串记为 $\Sigma^i$。例如，如果 $\Sigma=\{a,b\}$，那么 $L = \Sigma^2 = \{ab, ba, aa, bb\}$

然后我们定义一些基本操作

+ Concatenation: putting two strings together。如 $x=aa;y=bb;xy=aabb$
+ Power: concatenating multiple copies of a letter or word。如 $a^n=a·a^{n-1};a^1 = a; a^2 = a·a$，若 $x=ab;x^3=ababab$
+ Kleene Star: zero or more copies of a letter or word。如 $a^*=\{\varepsilon, a, aa, aaa, \dots\}$, $x=ab;x^*=\{\varepsilon, ab, abab, ababab, \dots \}$

## Finite-state automaton

接下来的定义部分为了避免翻译不准确带来的歧义，这里都使用英文原文。

A **finite-state automaton comprises the following elements:**

+ A sequence of **input symbols** (the input "tape")
+ The **current location in the input**, which indicates the current input symbol (the read "head")
+ The **current state of the machine** (denoted q0, q1,..., qn)
+ A **transition function** which inputs the current state and the current input, and outputs a new (next) state

在计算过程中

+ FSA 在初始状态开始（通常称为 q0）
+ 每一步，状态转移方程会接受当前的输入符号和当前的状态，并且更新到一个新的状态，指针指向下一个符号
+ 在 FSA 到达输入的末尾时，计算过程结束

一个或多个状态可以被标记为最终状态，只有计算停止在这些状态时，才认为是计算成功。更多的细节及符号形式请参考[这里](https://zh.wikipedia.org/wiki/%E6%9C%89%E9%99%90%E7%8A%B6%E6%80%81%E6%9C%BA)

### Regular Languages

![](/images/14564392411172.jpg)

### NDFSA 与 DFSA

实话说这部分不是特别看得懂，所以只能给出参考链接了

+ NDFSA - [非确定有限状态自动机](https://zh.wikipedia.org/wiki/%E9%9D%9E%E7%A1%AE%E5%AE%9A%E6%9C%89%E9%99%90%E7%8A%B6%E6%80%81%E8%87%AA%E5%8A%A8%E6%9C%BA)
+ DFSA - [确定有限状态自动机](https://zh.wikipedia.org/wiki/%E7%A1%AE%E5%AE%9A%E6%9C%89%E9%99%90%E7%8A%B6%E6%80%81%E8%87%AA%E5%8A%A8%E6%9C%BA)

## Pushdown Automaton 下推自动机

下推自动机比有限状态自动机复杂：除了有限状态组成部分外，还包括一个长度不受限制的栈；下推自动机的状态迁移不但要参考有限状态部分，也要参照栈当前的状态；状态迁移不但包括有限状态的变迁，还包括一个栈的出栈或入栈过程。

具体的解释仍然是比较抽象，这里也没办法展开了，详情参考[这里](https://zh.wikipedia.org/wiki/%E4%B8%8B%E6%8E%A8%E8%87%AA%E5%8A%A8%E6%9C%BA)

## Turing Machines 图灵机

接下来的定义部分为了避免翻译不准确带来的歧义，这里都使用英文原文。

+ The basic model of a Turing machine has a finite control, an input tage that is divided into cells, and a tape head that scans one celll of the tape at a time
+ The tage has a left most cell but is infinite to the right
+ Each cell of the tape may hold exactly one of a finite number of tape symbols
+ Initially, the n leftmost cells, for some finite n >= 0, hold the input, which is a string of symbols chosen from a subset of the tape symbols called the input symbols
+ The remaining infinity of cells each hold the blank, which is a special symbol that is not an input symbol

如下图所示：

![](/images/14564420943465.jpg)

具体的解释还是参考[维基](https://zh.wikipedia.org/wiki/%E5%9B%BE%E7%81%B5%E6%9C%BA)

## NP

最后是更加理论化的一个概念，非定常多项式（英语：non-deterministic polynomial，缩写NP）时间复杂性类，或称非确定性多项式时间复杂性类，包含了可以在多项式时间内，对一个判定性算法问题的实例，一个给定的解是否正确的算法问题。

具体请参考[维基](https://zh.wikipedia.org/wiki/NP_(%E8%A4%87%E9%9B%9C%E5%BA%A6))

