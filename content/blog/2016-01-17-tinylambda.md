---
title: TinyLambda
author: htfy96
type: post
date: 2016-01-17T00:49:50+00:00
url: /2016/01/17/tinylambda/
categories:
  - 代码

---
晚上突发奇想，想用C++实现一个无比简陋的lambda：只能作为匿名函数，不带闭包，无捕获，所有中间值和结果都是int，利用C++98实现。

这类lambda实际上就是Eigen这类库中延迟计算的一个原型，写起来并不复杂，大约100行左右代码就能完成。

测试例子：

<pre><code class="language-cpp">    // I don't like #define, but compatibility f**ks me every day
#define l (2 + _1 + _2) 
#define lnew  (_1 + 2 + 3 + _2)
#define l3  ((_1 + 2) &lt; _2)
    std::cout&lt;&lt; l(2,-43) &lt;&lt; std::endl;
    std::cout &lt;&lt; lnew(3,4) &lt;&lt; std::endl;
    std::cout &lt;&lt; l3(0,1) &lt;&lt; " "&lt;&lt; l3(-2, 1) &lt;&lt; std::endl;
    int a[] {1,4,2,6,7};
    std::sort(a, a+5, _1 &gt; _2);
    for(int i=0; i&lt;5; ++i)
      std::cout &lt;&lt; a[i] &lt;&lt; std::endl;
/*
-39
12
0 1
7
6
4
2
1
*/</code></pre>

# 分析

我们最终要实现的一个东西，我们希望它具有的特性是：

  * 具有`int operator() (const int, const int)`，能当作函数使用
  * 能在`+`、`<`运算符下和整数和自身的类运算，产生一个新的这类对象

之前我们分析过：对于这类抽象，如果需要运行时多态，就应该使用继承与虚函数；如果不需要，就应该使用[静态多态][1]。

## 动态多态

先尝试一下动态多态的思想：

定义接口：

<pre><code class="language-cpp">class lambda_interface
{
    protected:
        virtual lambda_interface* clone() const = 0;
    public:
        virtual ~lambda_interface() {};
        virtual int operator() (const int, const int) const = 0;
        friend add_lambda;
};
</code></pre>

对于`+`，返回一个继承自`lambda_interface`的子类：

<pre><code class="language-cpp">plus_lambda operator+ (const lambda_interface& a, const lambda_interface& b) //不能返回lambda_interface，否则类型信息会丢失
{
    return plus_lambda(a,b);
}

class add_lambda : public lambda_interface
{
    private:
        lambda_interface *p1, *p2;
    public:
        virtual int operator() (const int a, const int b) const
        {
            return p1-&gt;operator()(a,b) + p2-&gt;operator()(a,b);
        }

        add_lambda(const lambda_interface& m1, const lambda_interface& m2): 
            p1(m1.clone()), p2(m2.clone())
            {}    

        add_lambda(const add_lambda& a) : p1(a.p1-&gt;clone()), p2(a.p2-&gt;clone()) {}

        virtual lambda_interface* clone() const
        {
            return new add_lambda(*this);
        }

        virtual ~add_lambda()
        {
            delete p1; delete p2;
        }
};</code></pre>

这里需要注意的是clone，众所周知构造函数不能是虚函数，但是复制构造可以利用`clone()`来变相实现虚函数的效果。

定义两个`placeholder`，它们始终返回第1/2个参数：

<pre><code class="language-cpp">class placeholder1 : public lambda_interface
{
    public:
        virtual int operator() (const int a, const int b) const
        {
            return a;
            (void)(b);
        }

        virtual lambda_interface* clone() const
        {
            return new placeholder1(*this);
        }
        virtual ~placeholder1() {}
};
class placeholder2 : public lambda_interface
{
    public:
        int operator() (const int a, const int b) const
        {
            return b;
            (void)(a);
        }

        virtual lambda_interface* clone() const
        {
            return new placeholder2(*this);
        }
        virtual ~placeholder2() {}
};

placeholder1 _1;
placeholder2 _2;</code></pre>

对于常数，我们构造一个`constant`类把它包裹起来：

<pre><code class="language-cpp">class constant: public lambda_interface
{
    private:
        int n_;
    public:
        constant(const int m): n_(m) {}
        constant(const constant& m) : n_(m.n_) {}
        virtual int operator() (const int, const int) const
        {
            return n_;
        }
        virtual lambda_interface* clone() const
        {
            return new constant(*this);
        }
        virtual ~constant() {}
};</code></pre>

测试：

<pre><code class="language-cpp">#define l ( _2 + constant(2) + _1 )
cout &lt;&lt; l(2, 12) &lt;&lt; endl; // 16</code></pre>

迄今为止，我们已经实现了动态多态的`+`，我们发现，这一实现存在着诸多不足：

  * `+`中的常数暂时还不能自动转换成constant类型，需要手动包裹。之后的改进手段就是重载`+(const lambda_interface&, const int)`等
  * 较为丑陋，这其中所有信息都可以编译时获得，引入了虚函数增加了大量不必要的负担，并且中间堆上分配了不少内存，容易造成性能问题。

## 静态多态

静态多态不利用语法上的继承来实现`is_a`的特性描述，而是利用模板+各个类相同的接口来实现一种语法在不同类上不同的表现。

例如：

<pre><code class="language-cpp">struct A
{
    void print() { cout&lt;&lt;1&lt;&lt;endl; }
};

struct B
{
    void print() { cout&lt;&lt;2&lt;&lt;endl; }
};

template&lt;typename T&gt;
void print(const T& m)
{
    m.print();
}

//调用：
A a;
B b;
print(a); //1
print(b); //2</code></pre>

这样就实现了语义上的多态性。

实现：

这个不是接口，而是为了减少代码量而把共同部分抽出来了而已：

<pre><code class="language-cpp">template&lt;typename T1, typename T2&gt;
class lambda_base
{
    protected:
        T1 a1_;
        T2 a2_;
    public:
        lambda_base(const T1& a1, const T2& a2) : a1_(a1), a2_(a2) {}
};</code></pre>

因为一个运算符其两个操作数的类型没有统一的接口，所以利用模板来生成基类。

`+`类：

<pre><code class="language-cpp">template&lt;typename T1, typename T2&gt;
class lambda_add : protected lambda_base&lt;T1, T2&gt;
{
    public:
        lambda_add(const T1& a1, const T2& a2) : lambda_base&lt;T1, T2&gt;(a1, a2) {} //简单转发一下
        int operator() (const int arg1, const int arg2)
        {
            return lambda_base&lt;T1, T2&gt;::a1_(arg1, arg2) + lambda_base&lt;T1, T2&gt;::a2_(arg1, arg2);
        }
};</code></pre>

`+`函数也要利用模板来生成，注意：这里把`int`替换成了`constant`类：

<pre><code class="language-cpp">template&lt;typename T1, typename T2&gt;
lambda_add&lt;typename replace&lt;T1&gt;::type, typename replace&lt;T2&gt;::type&gt;
operator+ (const T1& a1, const T2& a2)
{
    return lambda_add&lt;typename replace&lt;T1&gt;::type, typename replace&lt;T2&gt;::type&gt;(a1, a2);
}</code></pre>

`replace<T>::type`如果`T`不是`int`，返回自身，否则返回`constant`

<pre><code class="language-cpp">template&lt;typename T&gt;
struct replace
{
    typedef T type;
};

template&lt;&gt;
struct replace&lt;int&gt;
{
    typedef constant type;
};</code></pre>

`constant`和`placeholder`:

<pre><code class="language-cpp">class constant
{
    private:
        int c_;
    public:
        constant(const int c) : c_(c) {}
        int operator() (const int arg1, const int arg2)
        {
            return c_;
            (void)(arg1); (void)(arg2); //eliminate warning
        }
};

class placeholder1
{
    public:
        int operator() (const int arg1, const int arg2)
        {
            return arg1;
            (void)(arg2);
        }
};

class placeholder2
{
    public:
        int operator() (const int arg1, const int arg2)
        {
            return arg2;
            (void)(arg1);
        }
};

placeholder1 _1;
placeholder2 _2;</code></pre>

静态多态版本完成（上面省略了`<`和`>`，和`+`一样处理就行了

<pre><code class="language-cpp">#define l (2 + _1 + _2) 
#define lnew  (_1 + 2 + 3 + _2)
#define l3  ((_1 + 2) &lt; _2)
    std::cout&lt;&lt; l(2,-43) &lt;&lt; std::endl;
    std::cout &lt;&lt; lnew(3,4) &lt;&lt; std::endl;
    std::cout &lt;&lt; l3(0,1) &lt;&lt; " "&lt;&lt; l3(-2, 1) &lt;&lt; std::endl;
    int a[] {1,4,2,6,7};
    std::sort(a, a+5, _1 &gt; _2);
    for(int i=0; i&lt;5; ++i)
      std::cout &lt;&lt; a[i] &lt;&lt; std::endl;
/****************
-39
12
0 1
7
6
4
2
1
*****************/</code></pre>

# 总结

以上全部代码的完整版都能在：<https://github.com/htfy96/CppToys/tree/master/lambdatest> 看到

静态多态在不需要运行时信息时，不仅降低了overhead，而且给程序编写带来了很大的简洁。在这种模式下，C++中的类型可以当作`duck typing`的方式来使用，从而给了程序员很大的自由。 不过，缺点也在于弱类型约束一旦出现bug很有可能会产生大量无用报错信息，缓解这一缺点可以使用`SFINAE`，或是进入C++11年代的`static_assert`(你都有了C++11了手写什么lambda)，或是未来C++17的`concept`。

总之，C++不强迫程序员选择什么，这就是为什么我非常喜欢这门语言。

# Self-promotion

SJTU大二本科生，寻求暑假实习中。偏向于后端，有利用Golang+redis开发的经历。

Linkedin：<http://cn.linkedin.com/in/vicluo>

 [1]: http://www.cnblogs.com/lizhenghn/p/3667681.html "静态多态"