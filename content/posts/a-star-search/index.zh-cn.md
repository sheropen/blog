---
weight: 1
title: "Informed Search: A* 搜索"
date: 2022-09-28T00:00:00+08:00
lastmod: 2022-09-28T00:00:00+08:00
draft: false
author: "Yu Jun Hao"
description: "A*搜索算法作为informed search的一种，能有效的对搜索树进行求解，同时确保解的全局最优性."
images: []
resources:
  - name: "featured-image"
    src: "featured-image.jpg"

tags: ["搜索问题", "算法"]
categories: ["documentation"]

lightgallery: true

toc:
  auto: false
math:
  enable: true
---

<!--more-->

### 简述

搜索问题可以转换为对搜索树的求解，求解方法基本如下：
<image src="tree-search.png">
可以看到，若根据不同的**strategy**从**candidates**中选择节点，就构成了不同的搜索算法。举例来说，深度优先算法（DFS）与广度优先搜索（BFS）就分别采用了**LIFO**和**FIFO**原则作为选择策略。

### Uniform-cost Search & Greedy Search

乍看之下，一致代价搜索（UCS）和贪心搜索都是选择代价最小的节点，但是二者对于代价的估值方向截然不同。UCS 总是选择从根节点开始到当前节点$x$，路径总代价$g(x)$最小的节点；而贪心搜索是通过启发式函数（heuristic function）对节点到目标节点的代价$h(x)$进行估值，并选择估值最小的节点。

相较而言，UCS 算法中只考虑 backward cost，因而虽然能保证全局最优性，但是算法性能不佳； 而贪心算法只考虑估算 forward cost，因此虽然算法性能高效，却十分依赖于$h(x)$的构造精度，并且往往只能保证局部最优性。

### A\* Search

A\*搜索综合了 UCS 以及贪心搜索的优点，既保证了算法的全局最优性，又能有效降低算法的复杂度。A\*搜索的代价函数为$f(x)=g(x)+h(x)$。因此，A\*搜索同时兼顾了 backward cost 与 forward cost，对搜索问题的刻画更为准确。

然而，A\*搜索需要在一定条件下才能确保全局最优性。若$h^*(x)$为节点$x$到目标节点的真实代价，则 A\*搜索要求$h(x)$为**admissible heuristics**，即：  
$$0\leq h(x)\leq h^\*(x)$$

换句话说，$h(x)$必须是乐观的，这样才能避免最优路径上的节点被$h(x)$拖累。试想想，若最优路径上某个节点的$h(x^*)=\infty$，那么该节点一定会留到最后才被选择。

算法的取舍点主要在于$h(x)$。当$h(x)$越接近$h^*(x)$，即$h(x)$描绘得越准确，那么算法就能越快找到最优路径。反之，$h(x)$提供的信息不足，则会造成算法对不必要的 bad path 进行搜索。当$h(x)=0$时，不能提供任何关于 forward cost 的信息，则 A\*搜索退化为 UCS。  
<image src="comparison.png" caption="迷宫问题的搜索过程，红色越亮代表节点越早被选择">

### Admissible Heuristics

构造 admissible heuristics 的要点在于考虑真实代价$h^*(x)$**至少**是多少。一个经常被用到的方法就是将问题简化，经简化后的**relaxed problem**的解往往是原问题的 admissible heuristics。  
<image src="admissible-heuristics.jpeg">  
考虑上述的迷宫问题，我们可以将问题简化为"**若 pac-man 可以穿越墙壁，那么到达终点的最短距离是多少？**"。问题的解显然是从起点到终点的曼哈顿距离。不难证明，对于有墙壁的原问题来说，从某个节点到终点**至少**也得走这么多步。因此曼哈顿距离是原问题一个很好的 admissible heuristics。

### A\*搜索 optimality 证明

<image src="a-star-optimality.jpeg">
欲证全局最优节点$A$必定比任意局部最优节点$B$先被选择。  
  
考虑局部最优节点$B$在待选择队列fringe里，而$A$不在（若$A$也在，则由$f(A)\le f(B)$得证），则必定有$A$的先驱节点$n$存在于fringe内。
  
1. $f(n) = g(n) + h(n)\leq f(A) = g(n) + h^*(n)$ 
2. $f(A) \leq f(B)$ （由$A$的全局最优性与$B$的局部最优性）

$\therefore f(n) \leq f(B)$ （由 ① 与 ②），则$A$的所有先驱节点$n$会比节点$B$先被选择，直到$A$也在 fringe 内。由$f(A)\leq f(B)$得证。

### Graph Search

在 UCS 中，我们可以**忽略所有曾经访问过的节点**以提高算法的效率却仍保证全局最优性，这是因为如果一个节点被访问过，那么该路径的代价$g(x)$一定不比当前路径的代价高。然而，在 A\*搜索中，我们若想这么做，就得加强对$h(x)$的约束。此时，A\*搜索要求相邻节点$s,s'$的 heuristics 满足**consistency**，即：
$$h(s)\leq cost(s\to s')+h(s')$$
直观上来看，**consistency**保证从节点$s$出发到终点的代价估值，绝对不会超过先从节点$s$到$s'$后，再从$s'$到终点的代价估值（最理想的状态下，从$s$到终点的最优路径确实途径$s'$，则此时$h(s)=cost(s\to s')+h(s')$）。一个[比喻](https://www.zhihu.com/question/23052955/answer/1648645823)是计程车司机对从上海到北京的路费估价，绝对不会比从上海到广州再到北京的估价高。

数学层面上，**consistency**保证了对于任意路径$(s_0,s_1,...,s_k)$，$f(s_i)\leq f(s_{i+1})$始终成立。证明如下：

$$
f(s_i) = g(s_i) + h(s_i) \leq g(s_i) + cost(s_i\to s_{i+1}) + h_{i+1} = f(s_{i+1})
$$

在 A\*搜索中，若想忽略曾经访问过的节点，我们最担心的一点就是某个处于最优路径的节点$n$，曾经作为节点$n'$被某些更差的路线访问过。利用反证法可证明这种情况不可能出现：
<image src="graph-search-optimality.jpeg">  
假设上述情况成立，则在$n'$被选择前，必定存在某个$n$的先驱节点$p$也在 fringe 内。此时，由**consistency**知$f(p)\lt f(n)$。同时，由$n'$非全局最优知$f(n)\lt f(n')$ 。则有：
$$f(p)\lt f(n')$$  
故$n$的所有先驱节点$p$都会比$n'$先被选择，直到节点$n$也在 fringe 内。此时应先选择节点$n$，与前提矛盾，证明成立。

{{< admonition info "补充知识" >}}
我们可以由[consistency 推出 admissible](https://en.wikipedia.org/wiki/Consistent_heuristic)，反之不然。
考虑任一节点$N_i$，若$N_0$为终点，且$(N_i,N_{i-1},...N_0)$为从$N_i$到终点的最优路径，则：

$$
h(N_i) \\\
\leq c(N_i\to N_{i-1})+ h(N_{i-1}) \\\
\leq...\\\
\leq c(N_i\to N_{i-1}) + c(N_{i-1}\to N_{i-2}) + ... + c(N_1\to N_0) + h(N_0) \\\
{?=}h^*(N_i)
$$

{{< /admonition >}}

### Credit

1. [Abbeel, P. & Klein, D. (2018, September 6). Lecture 3: A\* Search and Heuristics. Youtube.](https://www.youtube.com/watch?v=Mlwrx7hbKPs&ab_channel=WebcastDepartmental)
   {{< youtube Mlwrx7hbKPs >}}

2. [Foote, D. (2016, June 23). CS 188: A\* Graph Search Optimality. Youtube.](https://www.youtube.com/watch?v=AVKPExS4TBY&ab_channel=CS188)
   {{< youtube AVKPExS4TBY >}}
