---
title: parallel_accumulate
author: htfy96
type: post
date: 2016-02-06T06:14:11+00:00
url: /2016/02/06/parallel_accumulate/
categories:
  - 代码

---
Too young的一个小练手……

<pre><code class="language-cpp">#include &lt;iostream&gt;
#include &lt;chrono&gt;
#include &lt;thread&gt;
#include &lt;vector&gt;
#include &lt;cstdlib&gt;
#include &lt;algorithm&gt;
#include &lt;numeric&gt;

using namespace std;

vector&lt;int&gt; v;

template&lt;typename T, typename U&gt;
void MyAccuX(T beg, T ed, U& ans)
{
    ans = accumulate(beg, ed, 0);
}

template&lt;typename T, typename T3&gt;
T3 parallel_accu( T beg,  T ed, T3 initial)
{
    size_t d = distance(beg, ed);
    size_t part = thread::hardware_concurrency();
    size_t inteval = d / part;

    T tmp(beg);
    advance(tmp, inteval);

    vector&lt;thread&gt; vt;
    vector&lt;T3&gt; result(part);
    for(size_t i=0; i&lt;part; ++i)
    {
        vt.push_back( thread(MyAccuX&lt;T, T3&gt;, beg, tmp, ref(result[i])));
    }
    for(size_t i=0; i&lt;part; ++i)
    {
        if (vt[i].joinable()) vt[i].join();
    }

    for(size_t i=0; i&lt;part; ++i)
    {
        initial += result[i];
    }
    return initial;
}

int main()
{
    cout &lt;&lt; "Generating data" &lt;&lt; endl;
    v.reserve(500000000);
    for(long i=0; i&lt;500000000; ++i)
    {
        if (!(i%5000000)) cout &lt;&lt; i/5000000&lt;&lt;"%"&lt;&lt;endl;
        v.push_back(rand());
    }

    cout &lt;&lt; "begin"&lt;&lt;endl;
    auto t1 = chrono::high_resolution_clock::now();
    cout &lt;&lt; accumulate(v.begin(), v.end(), 0LL) &lt;&lt; endl; //4.18088
    //cout &lt;&lt; parallel_accu(v.begin(), v.end(), 0LL) &lt;&lt; endl; // 1.1092
    auto t2 = chrono::high_resolution_clock::now();
    cout &lt;&lt; chrono::duration_cast&lt;chrono::microseconds&gt;(t2-t1).count() / 1000000.0 &lt;&lt; endl;
}</code></pre>