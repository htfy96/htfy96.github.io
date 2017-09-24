---
title: C++ Magic Collection
author: htfy96
type: post
date: 2015-12-25T22:05:26+00:00
url: /2015/12/26/c-magic-collection/
categories:
  - 代码

---
嘛……最近这个博客好久没有写干货了。正好今晚<del>有空</del> 可以摸摸鱼，就来说说最近Cpp的一些东西吧。果然Cpp是世界上最复杂的语言（x

## R-value 运算符

C++11之后，可以利用

<pre><code class="language-cpp">return_t operatorX(arg_t) &;
return_t operatorX(arg_t) &&;</code></pre>

来针对左值和右值对象重载不同版本的运算符。典型例子就是

<pre><code class="language-cpp">T& opearator++() &;
T&& operator++() &&
{
    this-&gt;operator++();
    return std::move(*this);
}</code></pre>

如果不写左值和右值不同的版本，那么：

    T obj2(++std::move(obj1));

会退化到使用左值版本的构造函数，造成性能损失。

但是，在
  
<https://www.zhihu.com/question/38851602/answer/78421911?group_id=663734370451853312#comment-110278512>
  
有人提到了，根本就不应该对右值提供这类运算符，对此个人持保留意见。而且，对于`foo(++bar())`这类语法，如果使用`foo(bar() + 1)` `T tmp = bar(); ++tmp; foo(tmp);`等语法都会造成`foo`退化到左值版本，观望中。

## 前C++11年代的函数接口

众所周知，C++11引入了`std::function`，针对不同类型的可调用对象，提供了统一的接口。那么，在C++11之前，函数的接口是怎么样的呢？

> 警告：这部分内容在C++11中已经被标为deprecated, 并在C++17中正式被移除，因此请勿在工程中使用这部分内容 

因为没有参数包，所以函数接口肯定只能限制在有限个参数。而最有意义的接口有两个：映射函数`ret_t map(original_t)`和二元函数`ret_t comp(original1_t, original2_t)`。在C++98的年代，要想实现函数接口的多态性，有两种方式：

一种方式是定义一个原始的`ret_t (*FuncType)(original_t)`类型，通过给`FuncType`类型的变量赋不同函数指针的值，来实现函数接口的方式；

另一种就是(1)（动态绑定）构造一个继承自一个有`virtual ret_t operator()(original_t) = 0`类的Functor，然后定义这个类的`operator()`，保存父类，通过`operator()`的多态性来实现函数接口。或(2)（静态绑定）父类`typedef xxx arg_1_t; typedef yyy ret_t;`，然后子类继承父类实现`operator()`来提供函数接口，这些过程全部在编译期完成。

不难发现，Functor可以看作一个闭包，而函数指针不能，因此C++98采取了后一种策略，而且提供的是(2)的静态绑定。

对应两种接口，C++定义了以下两个父类：

<pre><code class="language-cpp">template &lt;typename ArgumentType, typename ResultType&gt;
struct unary_function
{
    typedef ArgumentType    argument_type;
    typedef ResultType result_type;
};

template&lt;
    class Arg1,
    class Arg2, 
    class Result
>struct binary_function
{
    typedef Arg1   first_argument_type;
    typedef Arg2   second_argument_type;
    typedef Result result_type;
};

// Usage:
struct Functor1 : public std::unary_function&lt;int, bool&gt;
{
    bool operator()(int x) { return x&lt;42; }
};
</code></pre>

有了这些接口之后，很容易就能定义出一些关于函数的操作：

<pre><code class="language-cpp">template&lt; class Predicate &gt;
struct unary_negate : public std::unary_function&lt;Predicate::argument_type, bool&gt;
{
    //...
    // Possible Implementation:
    Predicate predicate_;
    unary_negate(const Predicate &pred) :
        predicate_(pred) {]
    bool operator()(const Predicate::argument_type &x)
    {
        return predicate_(x);
    }
}; //对Unary取反
template&lt; class Predicate &gt;
struct binary_negate :
    public std::binary_function&lt;
        Predicate::first_argument_type,
        Predicate::second_argument_type,
        bool
    &gt;;

//这两个的函数版本：
template&lt; class Predicate &gt;
std::unary_negate&lt;Predicate&gt; not1(const Predicate& pred);
template&lt; class Predicate &gt;
std::binary_negate&lt;Predicate&gt; not2(const Predicate& pred);
// 实现太简单，自己想
</code></pre>

除此之外，为了方便从函数构造Functor, 标准库还提供了函数`ptr_fun`，接受一个`unary function` 或者一个`binary function`的函数指针，并返回一个对应`std::unary_function`或者`std::binary_function`的Functor：

<pre><code class="language-cpp">//Possible Implementation:

template&lt;typename Arg, typename Result&gt;
struct pointer_to_unary_function : std::unary_function&lt;Arg,Result&gt;
{
    Result (*p_)(Arg);
    pointer_to_unary_function(Result(*ptr)(Arg)) : 
        p_(ptr) {}

    Result operator()(Arg x)
    {
        return p_(x);
    }
};

template&lt;typename template&lt; class Arg, class Result &gt;
std::pointer_to_unary_function&lt;Arg,Result&gt;
    ptr_fun( Result (*f)(Arg) )
{
    return pointer_to_unary_function(f);
}
</code></pre>

普通的函数可以直接`not2`，因为函数指针本身也满足`()`运算符的约束。但要想把它装起来，还是需要`ptr_fun`才能做到，因为`binary_functon`没有函数指针的构造函数。然而，在静态绑定下，把`binary_function`装起来并没有什么用，因为它不具有`operator()`：

<pre><code class="language-cpp">bool compare(const MyType &arg1, const MyType &arg2); 

sort(xxx.first(), xxx.second(), not2(compare));

vector&lt;std::binary_function&lt;int, int, bool&gt; &gt; v;
v.push_back( ptr_fun(compare) );</code></pre>

`ptr_fun`产生的类叫做`pointer_to_binary_function<Arg1, Arg2, Result>`，就像一个非常原始的`std::function<Result(Arg1,Arg2)>`一样，是函数指针的简陋封装，比较naive。

然后为了支持成员函数指针，还拥有了一个叫做`mem_fun`的函数：
  
它的返回值是`mem_fun_t`，保存了一个成员函数指针。在使用时，`Result operator() (class_t*)`，需要传入一个对象的指针。同样，对于需要一个参数的成员函数，还有`mem_fun_1`函数和对应的`mem_fun_1_t`，构造方式大致相同.

简陋的绑定：只支持`binary_function`到`unary_function`的绑定，通过`binder1st`和`binder2nd`两个类模板来实现，非常的simple。

最后讨论一下C++98这种做法的不足：

  * 没有参数包，所以只内置了一元和二元的函数接口，其它的都要自己写
  * 只提供了静态绑定，功能受限
  * 绑定的时候不能绑定引用:<http://stackoverflow.com/questions/7822652/using-bind1st-for-a-method-that-takes-argument-by-reference>

因此，C++11的`std::function`是一个更加完善的东西。为了兼容性当然也可以使用`boost::function`。

写了这么多，都是在为下一篇《分析std::function》做准备，不过可能要在很久之后了。