---
title: 写点C++ user defined literals的奇怪（正常）用途
author: htfy96
type: post
date: 2017-01-08T12:17:48+00:00
url: /2017/01/08/写点c-user-defined-literals的奇怪（正常）用途/
categories:
  - 代码
tags:
  - c++
  - 小作品

---
&nbsp;

之前写的围棋程序的测试中，我们常常需要vector< { Player, Point } > pieceToPlace。问题在于，如果我们硬编码这个vector，可读性很差：

<pre class="language-cpp">pieceToPlace.emplace_back(Player::B, make_pair(2, 3));
pieceToPlace.emplace_back(Player::B, make_pair(2, 1));
pieceToPlace.emplace_back(Player::W, make_pair(1, 0));</pre>

通过User defined literal，我们可以做到这样：

<pre class="language-cpp">auto placeVec = getInput({
    {O,   1_B,   2_W,   4_W},
    {6_W, 3_B,   5_B,   7_W},
    {O,   O,     O,     8_B},
    {9_W, O,     O,     O}
});
for (auto &ele : placeVec) 
{
    cout &lt;&lt; (ele.player == Player::B ? "B" : "W") &lt;&lt; " " &lt;&lt; ele.point.first &lt;&lt; "," &lt;&lt; ele.point.second &lt;&lt; endl; 
}
/*
B 0,1
W 0,2
B 1,1
W 0,3
B 1,2
W 1,0
W 1,3
B 2,3
W 3,0
*/</pre>

<!--more-->

代码：

<div class="gist-oembed" data-gist="htfy96/3a5ef5ba544ae0beb3ca7bc38425e325.json">
</div>

&nbsp;

&nbsp;