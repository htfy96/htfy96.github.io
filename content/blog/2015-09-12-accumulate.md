---
title: accumulate
author: htfy96
type: post
date: 2015-09-12T05:10:05+00:00
url: /2015/09/12/accumulate/
categories:
  - 代码

---
### std::accumulate

<pre><code class="language-cpp">template&lt; class InputIt, class T &gt;
T accumulate( InputIt first, InputIt last, T init );
//(1)   
template&lt; class InputIt, class T, class BinaryOperation &gt;
T accumulate( InputIt first, InputIt last, T init,
              BinaryOperation op );
//(2)   </code></pre>

异曲同工。。。