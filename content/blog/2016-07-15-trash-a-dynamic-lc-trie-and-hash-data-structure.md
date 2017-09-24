---
title: TRASH A dynamic LC-trie and hash data structure
author: htfy96
type: post
date: 2016-07-15T04:54:17+00:00
url: /2016/07/15/trash-a-dynamic-lc-trie-and-hash-data-structure/
categories:
  - 代码
tags:
  - trie
  - 数据结构

---
http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.96.2143

传统Trie的优化。

  * path compression: 连续若干个只有一个孩子节点可以压成一个
  * LC（level compression）：若某一节点下面第k层的子树都非空，那么中间的k-1层都可以省略，直接把第k层的2^k个子树都接到自身 
      * 压缩原理：现代指针大小8byte远远大于char的大小
  * TRASH：在每个插入Trie的子串前面都加一个hash值，这样能在LC的基础上分布更均匀