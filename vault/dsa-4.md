title: 数据结构与算法 第 4 课 树
categories:
- Technique
toc: true
date: 2016-02-24 11:44:36
tags:
- CMU
- 数据结构
- 算法
---

从这一讲开始，慢慢就有点难度了，树的每个节点虽然看起来比较简单，但是不同结点以不同规则进行组合就使得难度指数级增长，后面的红黑树因为涉及到结点的调整，一定要按照前面提到的 5 个步骤来过一次，才能清晰理解。

<!-- more -->

---

## 树的应用

这次我们换个方式，先从不同的应用角度来了解一下树的用途。

**表示数学表达式**

我们可以用下面的树来表示 `b + a * b`

![](/images/14563638168396.jpg)

**XML Document Object Model**

XML 实际上也是一个树结构：

![](/images/14563638734277.jpg)

**表示迁移路径的概率**

下图可以认为，从 S 出发，有 p 的概率到 Su，有 1-p 的概率到 Sd

![](/images/14563639875028.jpg)

## 树的定义

下面介绍几种不同类型的树

### Free tree

指的是相互连接的无环无向图

![Free Tree](/images/14563645014537.jpg)

假设 G = (V, E) 是一个无向图，那么下面的语句是等价的：

1. G 是一个 free tree
2. G 中的任意两个节点间都有唯一的一条路径
3. G 是连通的的，但是如果去掉任何一条边，G 就不连通了
4. G 是连通的，并且 $|E|=|V|-1$
5. G 是无环的，并且 $|E|=|V|-1$
6. G 是无环的，但是加入任何一条新的边，就会变成有环的


### Forest 森林

同样是无环无向图，但不是所有的节点都是连通的

![Forest](/images/14563646007700.jpg)


下图中包含一个环，所以不能算是森林：

![包含环的例子](/images/14563646456272.jpg)

### Rooted Tree

有根的树也是一棵 free tree，但是其中一个节点和其他不同，称为根，下图中的 7 号节点就是根：

![](/images/14563698058000.jpg)

有根树中有一些概念需要理解清楚，这里只列举出来不再赘述：ancestor, descendant, proper ancestor, proper descendant, parent, child, siblings, external node(leaf), internal node。

一个节点的孩子数量称之为节点的度(degree)，从根到某节点的要经过的边的数量就是该节点的深度，最大的深度称为树的高度，如下图所示：

![](/images/14563702807914.jpg)

根树有几个特例，也非常常用，需要理解清楚，这里列举如下：

+ Binary tree
+ Full binary tree: each node is either a leaf or has degree exactly 2
+ Complete k-ary tree: a k-ary tree in which all leaves have the same depth and all internal nodes have degree k
+ Binary search tree: 一个节点的左右子节点和节点本身满足一定的大小关系

![Complete Binary Tree](/images/14563707343954.jpg)

### Catalan Numbers

先来看公式

![](/images/14563710814940.jpg)

然后我们就发现，Catalan 数对应的是有 n 节点的二叉树的数量（根据左右子树的位置可以有不同的结构）

## 树的遍历

这一部分也是非常重要的内容，基本来说各类考点都在这里，不但需要对概念的清晰理解，还需要利用递归来解决问题（虽然不用递归也可以），主要有下面这四类：

1. 前序遍历
2. 中序遍历
3. 后序遍历
4. 层次遍历

具体的概念可以在 wiki 上查看，注意一下层次遍历可能需要一些特殊处理即可。

## B-Tree

如果我们想要表示一个 complete binary tree 的话，其实可以用数组来完成，这种结构其实也可以看成是一个堆。堆的话需要理解的不算特别多，注意下最大堆最小堆，以及对应的操作即可。另外前面提到的优先队列也可以认为是堆，不过这个不展开了。

这一部分我们着重来看看 B-Tree，这是一类搜索树，在给定 n 个节点的条件下，尽可能减少树的高度，相对于原来的二叉搜索树，B-Tree 做了两个调整：

1. 节点可以有多于两个子节点
2. 节点中可以保存多个元素

因为需要保存不定数量的元素，所以一般用 set 来实现（这种情况下不允许有重复的元素，重复的情况这里暂时不考虑）。稍微提一下，许多数据库都是用 B-Tree 实现的。

所有的 B-Tree 都有一个非常重要的常数 MINIMUM，决定了每个节点中需要保存多少元素，具体的规则如下：

1. 根节点有 0 或 1 个元素，其他的节点至少需要保存 MINIMUM 个元素
2. 一个节点中最多可以保存 `2*MINIMUM` 个元素
3. 一个节点中保存的元素是有序的，从最小到最大
4. 假设一个非叶节点中存有 N 个元素，那么它会有 N+1 个子树
5. 对于任何一个非叶节点：
    + 第 I 个元素比其第 I 个子树的所有元素都要大
    + 第 I 个元素比其第 I+1 个子树的所有元素都要小
    + ![](/images/14563744343036.jpg)
6. 每个叶节点都有相同的深度，也就是说 B-Tree 总是平衡的

下图是一个例子，其中 MINIMUM = 1，注意，根节点的每个子节点也是一颗 B-Tree

![B-Tree 的例子](/images/14563745325794.jpg)


对应的数据结构是：

```java
public class IntBalancedSet{
    // constants
    private static final MINIMUM = 200;
    private static final MAXIMUM = 2 * MINIMUM;
    // info about root node
    int dataCount;
    int[] data = new int[MAXIMUM + 1];
    int childCount;
    // info about children
    IntBalancedSet[] subset = new IntBalancedSet[MAXIMUM+2];
    // ....
}
```

例如：

![](/images/14563980237017.jpg)

利用 B-Tree 进行搜索的方法如下：

+ 找到这样一个 set，满足 `data[I] >= target` 且 I 尽可能小，如果找不到，则 `I = dataCount`
+ 如果 `data[I] == target` 返回 true，如果不等于且没有子节点，返回 false
+ 如果不等于但是有子节点，则返回 `subset[I].contains(target)`

![利用 B-Tree 进行搜索的例子](/images/14563990361014.jpg)

一般来说，结构比较复杂的数据结构，进行修改都会比较麻烦（因为结构中内在的约束太多，变动的话需要满足所有约束），在 B-Tree 中添加和删除节点是比较复杂的操作。这里讲详细一些，用一个具体的例子来做说明（MINIMUM=1）

删除操作的伪代码：

```
Delete(T, X, success)
    // 从树 T 中删除 key 为 X 的节点，如果没有对应节点，则操作失败
    // 操作结果可以从 success 变量中获悉
    
    // 先找到 key 为 X 的节点的位置
    IF I is present THEN
        swap item I into leaf L which contains the inorder successor of I
        // 从 leaf L 开始删除操作
        IF L has no items THEN Fix(L)
        success := true
    ELSE
        success := false
```

然后是其中的 Fix 函数，用来处理没有子节点的节点，保证符合 B-Tree 的基本性质

```
Fix(N)
    // N 是一个没有子节点的节点
    // 如果 N 是一个 internal node，那么它有一个子节点
    
    Let P be the parent of N. If N is the root, delete it and return.
    
    IF some sibling of N has two items THEN
        distribute items among N, the sibling, and P
        
        IF N is internal THEN
            move the appropriate child from the sibling to N
    ELSE
        // 如果其兄弟没有两个子节点，就必须要进行融合
        Choose an adjancent sibling S of N
        
        Bring the appropriate item down from P into S
        
        IF N is internal THEN
            Move N's child to S
        
        Delete node N
        
        IF P is now without an item THEN Fix(P)
```

然后我们看看插入操作：

```
Insert(T, newitem)
    // 把 newitem 插入到树 T 中
    Let X be the search key of new item
    
    Locate the leaf L in which X belongs
    
    Add newitem to L
    
    IF L now has three items THEN
        Split(L)
```

继续来看看这里的 Split 函数

```
Split(N)
    // 分割那些有 3 个子节点的节点。注意如果 N 是 internal 的，它可以有 4 个孩子
    Let P be the parent of N
        // 如果 N 是根，那么创建一个新节点 P
        
    Replace node N by two nodes, N1 and N2
    
    Give N1 the item in N with the smallest search key value
    Give N2 the item in N with the largest search key value
    
    IF N is an iternal node THEN
        N1 becomes the parent of N's two leftmost children
        N2 becomes the parent of N's two rightmost children
    
    Send up to P the item in N with the middle search key value
    
    IF P now has 3 items THEN
        Split(P)
    
```

我们给定如下一颗 B-Tree

![](/images/14564007797015.jpg)

插入 39 之后（插入总是在叶节点）为

![插入 39 之后](/images/14564008049100.jpg)

再插入 38，会发现有一个节点有多于 2 个子节点，需要 split

![插入 38，split 之前](/images/14564008689598.jpg)


可以看到我们通过 split 操作把 39 提上去了（注意对照前面的 split 函数的伪代码来进行操作和理解）：

![插入 38，split 之后](/images/14564009041532.jpg)

我们再插入 37 与 36，这会导致树高改变，需要进行更多操作，过程如下：

![插入 37 与 36](/images/14564011447526.jpg)

插入之后我们发现又出现了非法的节点，依照 split 规则更新之后，上层的节点再次非法，这里属于有四个子节点的情况，左右分开，把中间的 37 移上一层，就成为最后一个形态。

我们再多插入一些，比如 34，35，36，整个过程如下：

![](/images/14564014630573.jpg)

最后就得到

![](/images/14564015026747.jpg)

注意，因为向下传播的关系，高度是从顶层开始增长的。

然后我们来看看删除操作，还是用刚才的例子，给定：

![](/images/14564016695179.jpg)

这次我们删除 50，因为是根节点，所以需要找一个节点放到原来根的位置，并且需要把 70 下移以满足条件

![删除 50](/images/14564016869839.jpg)

我们再删除 100 试试看：

![](/images/14564018498766.jpg)

同样对照前面的 Fix 函数进行操作即可。然后我们再删除 60，这里需要涉及的变动就会比较多，如下：

![](/images/14564020922018.jpg)

然后我们再删除 70 看看，就会变成这样：

![删除 70](/images/14564021367330.jpg)

最后再删除 80，又会进行一次融合

![删除 80](/images/14564021619408.jpg)

删除操作总是从 leaf 开始的，把要删除的节点和其 inorder successor 换位置。具体的操作如果还不明白，可以找一些视频来看看，这里不赘述了。

## 红黑树 Red Black Tree

这里只给出基本性质的说明，具体的细节可以查看[这里](https://zh.wikipedia.org/wiki/%E7%BA%A2%E9%BB%91%E6%A0%91)

红黑树是每个节点都带有颜色属性的二叉查找树，颜色为红色或黑色。在二叉查找树强制一般要求以外，对于任何有效的红黑树我们增加了如下的额外要求：

1. 节点是红色或黑色。
2. 根是黑色。
3. 所有叶子都是黑色（叶子是NIL节点）。
4. 每个红色节点必须有两个黑色的子节点。（从每个叶子到根的所有路径上不能有两个连续的红色节点。）
5. 从任一节点到其每个叶子的所有简单路径都包含相同数目的黑色节点。

![红黑树的例子](/images/14564058100039.jpg)


这些约束确保了红黑树的关键特性：从根到叶子的最长的可能路径不多于最短的可能路径的两倍长。结果是这个树大致上是平衡的。因为操作比如插入、删除和查找某个值的最坏情况时间都要求与树的高度成比例，这个在高度上的理论上限允许红黑树在最坏情况下都是高效的，而不同于普通的二叉查找树。

还是那句话对于约束较多的数据结构，进行插入和删除都是比较复杂的操作，最好通过例子掌握整个过程。

最后提一下复杂度，因为树的分叉设计，所以基本来说复杂度都是 log 的，但是需要注意，在最坏情况下，复杂度很可能是 O(n)，

