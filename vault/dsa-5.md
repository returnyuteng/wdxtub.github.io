title: 数据结构与算法 第 5 课 图
categories:
- Technique
toc: true
date: 2016-02-24 11:45:48
tags:
- CMU
- 数据结构
- 算法
---

前面我们介绍了几种不同类型的树，这一课我们在原来的基础上延伸一下，就能得到图的概念。相对于树，因为不同节点之间的关系更多，无论是表达形式或者是算法细节都会复杂不少，就更需要仔细揣摩。不过究其根本，实际上可以看做是不同矩阵间的信息检索，一定要注意细节。

<!-- more -->

---

## 基本介绍

图的话，从类别来看可以分为有向图与无向图，而最关键的操作，图的遍历，有深度优先（递归或者栈）和广度优先（队列）。

图由两类元素组成，一是节点(vertex)，二是边(edge)，下面是一个无向图的例子：

![无向图，边和节点都有对应标记](/images/14564071906087.jpg)

无向图和有向图都是一个有限的节点集合加上一个有限的边集合，每条边连接两个节点。不同的地方在于，无向图中连接的顺序是无关紧要的，而有向图中连接的顺序是重要的，是从 source 到 target 的方向，下面是一个有向图的例子：

![有向图的例子，注意箭头的方向](/images/14564073738319.jpg)

其他一些需要知道的术语：

+ Loops: edges that connect a vertex to itself
+ Paths: sequences of vertices p0, p1, ..., pm such that each adjacent pair of vertices are connected by an edge
+ Multiple Edges: two nodes may be connected by >1 edge
+ Simple Graphs: have no loops and no multiple edges

## 图的表示

这里介绍三种常见的方式，其实还有其他几种自定义的表示法，这里不拓展了

### 邻接矩阵 Adjacency Matrix

用一个二维矩阵来表示点和点之间的连接关系，如果一个图有 N 个节点，那么这个矩阵就是 NxN 的，对于节点 I 和节点 J，如果有一条边连接他们，那么第 I 行第 J 列的值为 true，否则是 false（如果是无向图的话，第 J 行第 I 列的值需要保持一致，也就是说以对角线为轴对称），下面是一个有向图的例子：

![有向图的邻接矩阵](/images/14564128326067.jpg)

### 边列表 Edge Lists

另外一种表达方式是每个节点有一个链表，表示它所连接的其他节点，如下图所示：

![](/images/14564132564620.jpg)

### 边集 Edge Sets

Edge Lists 中我们主要存储的是节点的信息，而在 Edge sets 中我们保存的是边的信息，包含每条边的 source 和 target。

### 性能分析

这里主要看一下最坏情况

+ 添加或删除边：
    + adjacency matrix: O(1)
    + edge list: O(N)
    + edge set: O(logN) - 使用 B-Tree
+ 检查某条边是否存在：
    + adjacency matrix: O(1)
    + edge list: O(N)
    + edge set: O(logN) - 使用 B-Tree 或 红黑树
+ 遍历某个节点的边：
    + adjacency matrix: O(N)
    + edge list: O(E) - 其中 E 是边的数目
    + edge set: O(E) - 使用 B-Tree 或 红黑树

## 图的实现


```java
public class Graph {
    private boolean[][] edges;
    private Object[] labels;
    
    public Graph (int n){
        edges = new boolean[n][n];
        labels = new Object[n];
    }
    
    public void addEdge (int s, int t) {
        edges[s][t] = true;
    }
    
    public Object getLabel(int v) {
        return labels[v];
    }
    
    public boolean isEdge(int s, int t) {
        return edges[s][t];
    }
    
    public int[] neighbors(int v) {
        int i;
        int count;
        int[] answer;
        count = 0;
        for (i = 0; i < labels.length; i++{
            if (edges[v][i]) count++;
        }    
        answer = new int[count];
        count = 0;
        for (i = 0; i < labels.length; i++) {
            if (edges[v][i]) answer[count++] = i;
        }
        return answer;
    }
    
    public void removeEdge(int s, int t) {
        edges[s][t] = false;
    }
    
    public void setLabel(int v, Object n){
        labels[v] = n;
    }
    
    public int size() {
        return labels.length;
    }
}
```

## 图的遍历

这个是图问题中比较常见的，一般来说，是从一个特定的节点开始，找到所有可能到达的节点。图和树的区别在于，我们可能会遇到环，所以就需要一种方法来处理这种情况。

我们可以用一个另外的数组来记录一个节点是否被访问过。

+ 深度搜索 - Stack
+ 广度搜索 - Queue

这一部分比较基础，大家可以自己找一个例子来边输出边测试，这里不再赘述。

## 最短路径

最短路径一般应用在有权有向图，如下所示：

![](/images/14564267263106.jpg)

我们要找的是路径最短的，但不一定是经过的边最少的，所以 BFS 在这里就不适用了。

我们可以使用 [Dijkstra's 算法](https://zh.wikipedia.org/wiki/%E6%88%B4%E5%85%8B%E6%96%AF%E7%89%B9%E6%8B%89%E7%AE%97%E6%B3%95)来处理，wiki 中有比较详细的说明，这里不再赘述。

对于比较稀疏的图，我们用堆来表示图会更好；反之，用邻接矩阵比较好。

另外一个利用动态规划的解法是 [Floyd Warshall 算法](https://zh.wikipedia.org/wiki/Floyd-Warshall%E7%AE%97%E6%B3%95)

索引中的各类算法也可以对应了解一下：

![](/images/14564271316392.jpg)


## 最小生成树 Minimum Spanning Tree (MST)

我们一般用 [Prim's 算法](https://zh.wikipedia.org/wiki/%E6%99%AE%E6%9E%97%E5%A7%86%E7%AE%97%E6%B3%95)，来寻找最小生成树，是一个贪心算法
 

