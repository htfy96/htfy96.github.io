---
title: 'C++: A Learner’s Guide'
author: htfy96
type: post
date: 2016-01-22T05:05:06+00:00
url: /2016/01/22/c-a-learners-guide/
categories:
  - 代码

---
> 这篇文章是写给即将开始C++学习的学弟学妹们的，对于C++入门者应该也有一定参考价值。 

* * *

今天有人问我如何学习C++，虽然我是Cpp <del>期末8X</del> 的蒟蒻，但是还是想写一点东西来说一说这件事情。

## 和Python的不同

大家都在第一学期学习过Python语言。不得不说，Python是一门非常适合初学的一门语言：有良好的标准库，语法简单，隐藏了很多细节。但是，为什么还要学习C++呢？

  * **贴近底层**
  
    实际上，提供给程序的内存只是连续的一个个地址。在Python中，你不需要太关心内存从哪里来，到哪里去。但是C++需要。这就要求学习中弄清楚C++的对象模型。C++设计的初衷就是**最小抽象**：在**满足要求**的前提下提供最小程度的抽象，这与很多现代语言差别很大。
  * **提供了更丰富的语法** 虽然表面上来看，C++比起Python语法更加严格，经常会报编译错误，但是实际上，C++提供的语法基础设施比Python丰富很多。不同于Python的“做一件事情有且仅有一种方法”（尽管实际上是95%的时间提供给你了1种正确方法，剩下5%时间没有方法），C++推崇的是“做一件事情有很多种方法，但是最好的仅有一种”（尽管给你的95%的方法都是问题频发、不正确的，4%的方法是正确的，1%的方法是优秀的）。这反映出C++需要大量的积累才能摸索出最佳实践。
  * **效率，还是效率** Python的动态类型决定了其速度的上限。在性能密集型而且又需要抽象的场合，C++仍然是唯一选择。（Source：<https://days2011.scala-lang.org/sites/days2011/files/ws3-1-Hundt.pdf>）
  * **学习多种编程范式** C++不仅仅是面向对象范式的体现，还包括程序编程和泛型编程。这多种范式给了C++多变的姿态以适应各种场合的要求，却也使得其成为了最复杂的语言之一。反之Python虽然也具有多种范式，但是Python的范式有所阉割（例：lambda表达式）或你们没有涉及到。

## 如何学习C++

  * **选择一本好书及之后的路线图** 学校发的那本书内容不全，流于表面，把C++当作C with class来教，而且还有一些错误。想认真学习C++的同学请认真读《C++ Primer》（第5版）（不是Plus）。可以说它才是C++的一本比较全面的介绍。</p> 
    在认真阅读完这本书之后，接下来这些都是很必要的：
    
      * 阅读Stackoverflow上C++话题下的前30页问题与答案：<http://stackoverflow.com/questions/tagged/c%2b%2b?sort=votes&pageSize=15> 
      * 《Effective C++》，《More Effective C++》。关注最佳实践。看完这两本书之后会发现之前写的一些语法上看似没有问题的程序问题多多。
      * 《深入了解C++面向对象模型》C++的对象如果一直当作黑箱理解肯定会出大乱子。这本书尽管部分内容有些过时，但是不多的关注底层的好书。
      * 《Effective Modern C++》这本书与Effective系列类似，但关注的点是C++11之后出现的新的语法的最佳实践。
      * 《C++语言的设计和演化》 C++之父著。一部关于妥协与进化的历史。「
      * 《C++FAQ》从设计者的角度解释为什么要这么设计。
      * 《C++FQA》批判《FAQ》中那些文过饰非的内容。和上面一条对比阅读。
    
    这是我个人的推荐，完整版可以查看http://stackoverflow.com/questions/388242/the-definitive-c-book-guide-and-list 。 </li> 
    
      * **有用的资源** 在学习中难免会碰到自己不懂的东西。这些东西应该到哪里去查询呢？以下是一些有用的资源：</p> 
          * 推荐环境：Orwell Dev C++，<http://sourceforge.net/projects/orwelldevcpp/> 单文件简单好用，自带的GCC4.9版本较新bug较少。
          * 文档查询：<http://en.cppreference.com/w/Main_Page> 排版清晰，内容权威
          * 本地文档查询：<https://zealdocs.org/download.html> 下载后选择Option-Docset-C++-Download下载完后就能本地查询了
          * 问题：<http://stackoverflow.com/> 先在上面用英文描述你的问题搜索，确认没有之后再提问。
      * **了解一些基本的常识**
  
        这里罗列一些课上基本不会说的，但是非常必要的常识，其中有一些可能需要在学一段时间之后再了解：</p> 
          * C++有哪几个版本的标准？它的特征有哪些？（阅读Wikipedia C++）
          * 它和C语言的关系是什么？(Stackoverflow: Relationship between C and C++)
          * 什么是声明？什么是定义？ 什么是编译？什么是连接？（C++ Primer）
          * 在如何编译一个项目？(Level1：编译https://github.com/olafurw/ML2048，试试能不能玩上2048。 Level2：搜索并学习Makefile以及Cmake，了解并尝试使用它们）
          * 学会使用git与github。（<http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000>）
      * **练习，还是练习**
  
        这里给出一些可以进行的练习：</p> 
          * 不使用STL库中的`map`等结构，实现一个K-V数据库(<https://en.wikipedia.org/wiki/Key-value_database>), 支持`is_exist(key)`, `add(key, value)`, `remove(key)`, `get(key)`这几种操作。
          * `std::vector`，请参照C++Reference实现一个完整的vector，不要少掉任何一个函数。
          * 实现一个General Problem Solver（<http://ai-su13.artifice.cc/gps.html>）
          * 实现简易的 $$$lambda $$$ 表达式。固定2个参数`_1`, `_2`，所有中间结果都是`int`，只需要支持`+`与`*`： 
            <pre><code class="language-cpp">#define lambda1 (_1 * 2 + _2)
cout &lt;&lt; lambda1(2,3) //7</code></pre>
        
          * (**)在C++11标准下，实现`std::function`。
      * **阅读优秀代码**
  
        思而不学则殆。在学习过程中需要阅读大量优秀的代码才能保证输入。</p> 
          * 标准库的`<vector>`、`<memory>`与`<function>`
          * google的leveldb：<https://github.com/google/leveldb> 工业级别代码
          * Boost库：<http://www.boost.org/doc/libs/> 注意不是去阅读源代码（因为黑科技太多，代码质量也参差不齐），而是了解有哪些优秀的库以及“C++还能这么用”的东西。推荐：`spirit`, `phoenix`, `mpl`, `preprocessor`, `compressed_pair`</ul> 
    
    最后，希望能竭尽所能掌握这一门神奇的语言。毕竟之后的语言千千万万，能当作出门语言的并不多。衷心祝愿大家能从此踏上代码的漫漫征途。