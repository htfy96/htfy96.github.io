---
title: Network_algo
author: htfy96
type: post
date: 2016-01-29T06:55:09+00:00
url: /2016/01/29/network_algo/
categories:
  - 代码

---
## 动机

实验室需要实现一些网络分析方面的算法。但是在写的时候出现了两个问题。

  * 一是数据库针对图的操作接口不统一。原先我们的数据库因为比较小是直接利用MySQL存一个NodeList和一个EdgeList表，之后数据量变大了一些之后将切换到Neo4j，难道这个算法要写两遍？之后如果更换成别的数据库的话会更麻烦。
  * 二是数据库在学校，而运行算法的程序在本地跑。两者之间的延时是需要考虑的一个因素，而且也不能预先加载出所有需要的结点。

因此，查阅了一些背景资料。

## 资料

包含图存储功能的解决方案有两类：

  * 一类是仅提供存储功能以及单点查询-单边查询的裸KV（SQL当成KV用其实也算这类）
  * 第二类是提供存储功能并提供类SQL DSL的数据库（有的还有计算功能）。例：a. Neo4j的Cipher([<http://neo4j.com/developer/cypher/>][1])，b. InfiniteGraph([[http://wiki.infinitegraph.com/3.4/w/index.php?title=PQL/Format\_of\_Operator_Expressions][2]][2])、MS的GraphEngine（[<http://www.graphengine.io/docs/manual/Utilities/geconfig.html>][3]）。其中，Neo4j的查询方式是通过socket通信直接发送Cipher串，而其余两者是自创的协议（文档中没有详细介绍）。

那么，我当前的问题其实可以扩展在1与2a上的情况，因为对于本身存储与计算强耦合的数据库，设计接口不仅可能损失性能，而且也比较繁琐。而且，2b没有提供成熟的非Java接口，所以开发会有一定难度。

针对网络分析的算法，<http://snap.stanford.edu/snap/quick.html> 已经实现了很多，但是这些没有针对远程查询针对特殊优化。

## 问题描述

设计一种能同时适应KV类图存储接口、DSL类接口的有关于网络操作的远程统一接口。要求：1)本地缓存、网络优化 2)利用最快速方式。（例：查询一个点的2度邻居信息在Neo4j中可以一条语句完成，但裸KV不行，mysql可以利用join加快一些速度）

同时，利用这个接口实现一些网络分析的算法（<del>不包含在SNAP中的最新算法</del>）。

## 设计

### 接口分析

对于一个图的查询来说，究竟什么样的接口才是正确的呢？不妨从需求来分析：

  * 本地缓存与网络优化就意味着查询接口需要支持批量查询。
  * 要利用最快速的操作方式就需要保证快速操作不能退化。

而Neo4j本身的操作就支持批量查询，同时，它支持的操作也非常典型。所以，我们希望图的接口能设计成Neo4j的形式，针对不同的目标数据库决定是否退化。

<del>同时，为了支持SNAP中已有的算法，这个图的接口还要保持与SNAP兼容。</del> (SNAP是MSVC ONLY，代码也非常陈旧了，算法没有为RemoteGraph单独设计，我们需要实现的算法很少依赖于它），而`boost::graph`带来的依赖管理会非常头疼，其中的算法也并非是特殊设计的，因此我们将自己构建一个图的库。

### 工作

  * (TODO) 首先写一个Cipher语言的Parser吧，只需要执行查询操作就可以了。 <del>我开始怀念编译原理了……</del>
  
    To be continued&#8230;

 [1]: http://neo4j.com/developer/cypher/
 [2]: http://wiki.infinitegraph.com/3.4/w/index.php?title=PQL/Format_of_Operator_Expressions
 [3]: http://www.graphengine.io/docs/manual/Utilities/geconfig.html