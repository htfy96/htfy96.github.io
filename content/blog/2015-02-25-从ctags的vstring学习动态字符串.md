---
title: 从cTags的vString学习动态字符串
author: htfy96
type: post
date: 2015-02-25T03:16:37+00:00
url: /2015/02/25/从ctags的vstring学习动态字符串/
categories:
  - 代码

---
在ctags中，vString分布在`vString.h`和`vString.c`两个文件中，代码非常简洁，加起来仅有300余行，是初学c语言与动态字符串一个很好研究的范例。

# 结构定义

在vString.h中，vString定义如下：

<pre><code class="language-c">struct sVString {
        size_t  length;  /* size of buffer used */
        size_t  size;    /* allocated size of buffer */
        char   *buffer;  /* location of buffer */
};</code></pre>

这也差不多是绝大多数动态字符串的实现方式。

`buffer`存储内容，`size`存储实际内容长度，按理说这样两个成员已经可以起到vString的作用了，但是为了速度起见，还是增加了`length`变量，使得每次增加新内容时不是分配一个单位大小，而是到达上限后一次分配多个单位。
  
宏

为方便起见，在定义下面的函数的时候，如果一个参数是vString，另一个参数可以是vString也可以是char _的，统一只定义了vString版本，因为vString.buffer本身就是一个传统的char_ 字符串。在这里通过宏，构造出char* 版本：

<pre><code class="language-c">#define vStringValue(vs)      ((vs)-&gt;buffer)
#define vStringCat(vs,s)      vStringCatS((vs), vStringValue((s)))</code></pre>

## exxx

凡是带exxx的函数(如eFree)，实际上是在原函数（如Free）上加上了相应的检查，如果有错误就报错，这种技术在sqlite中也有使用。

## xXXX

带x前缀的函数是自己实现的方便书写的宏。

<pre><code class="language-c">#define xMalloc(n,Type)    (Type *)eMalloc((size_t)(n) * sizeof (Type)) //分配n个Type类型并初始化
#define xCalloc(n,Type)    (Type *)eCalloc((size_t)(n), sizeof (Type))
#define xRealloc(p,n,Type) (Type *)eRealloc((p), (n) * sizeof (Type))</code></pre>

## DebugStatement(x)

<pre><code class="language-c">#ifdef DEBUG
# define DebugStatement(x) x
#else
# define DebugStatement(x)
#endif</code></pre>

括号内的语句只有在定义了DEBUG宏时才会被编译，这样对于调试单条语句减少了代码量。

# 函数定义

## vString *vStringNew (void);

新建一个vString并返回指针。

<pre><code class="language-c">vString *vStringNew (void)
{
    vString *const string = xMalloc (1, vString);
    string-&gt;length = 0;
    string-&gt;size   = vStringInitialSize;
    string-&gt;buffer = xMalloc (string-&gt;size, char);
    vStringClear (string);
    return string;
}</code></pre>

`vStringInitialSize`是32，就像标准库sort的那个常数一样是实验出来的。太大占用空间，太小开始时刻需要频繁分配内存影响速度。

vString *const，后置const不允许修改指针本身，涉及到指针操作在定义时就应该给予其最小的权限，这样即使误操作也有可能被编译器检查出来。

## void vStringClear (vString *const string)

清空vString。

<pre><code class="language-c">void vStringClear (vString *const string)
{
    string-&gt;length = 0;
    string-&gt;buffer [0] = '\0';
    DebugStatement ( memset (string-&gt;buffer, 0, string-&gt;size); )
}</code></pre>

以char*的通常规范，首位填\0，长度清空即可。注意这里clear的是buffer的大小，对字符串本身的size没有变动。默认情况下不用buffer清零，因为之后使用该部分时必然是先赋值的。

## void vStringSetLength (vString *const string)

自动根据buffer中实际内容大小设置size。

<pre><code class="language-c">void vStringSetLength (vString *const string)
{
    string-&gt;length = strlen (string-&gt;buffer);
}</code></pre>

使用strlen，避免重复造轮子。

## void vStringDelete (vString *const string)

彻底清空vString空间。

<pre><code class="language-c">void vStringDelete (vString *const string)
{
    if (string != NULL)
    {
        if (string-&gt;buffer != NULL)
            eFree (string-&gt;buffer);
        eFree (string);
    }
}</code></pre>

预先检查，防止重复清空。

## void vStringResize (vString *const string, const size_t newSize)

给buffer分配新的大小。

<pre><code class="language-c">void vStringResize (vString *const string, const size_t newSize)
{
    char *const newBuffer = xRealloc (string-&gt;buffer, newSize, char);

    string-&gt;size = newSize;
    string-&gt;buffer = newBuffer;
}</code></pre>

否则可能会发生实际给buffer赋值时赋值到了未定义区段，造成程序崩溃或内存被篡改等问题。

realloc有可能新的指针和旧指针一样，也有可能不同。

## boolean vStringAutoResize (vString *const string)

精华。决定了一次分配多大空间。

<pre><code class="language-c"> boolean vStringAutoResize (vString *const string)
{
    boolean ok = TRUE;
    if (string-&gt;size &lt;= INT_MAX / 2)
    {
        const size_t newSize = string-&gt;size * 2;
        vStringResize (string, newSize);
    }
    return ok;
}</code></pre>

如果有空间则分配当前空间的两倍。

这个也属于一些经验之道，因为一般来说，一个字符串本身的大小越大，它需要更大空间可能性越大，这个方法也在这里使用。

## void vStringPut (vString *const string, const int c)

新增加一个字符。

注意到由于这个函数用得非常频繁，为了减少调用开支，定义了一个宏版本。

<pre><code class="language-c">void vStringPut (vString *const string, const int c)
{
    if (string-&gt;length + 1 == string-&gt;size)  /*  check for buffer overflow */
        vStringAutoResize (string);

    string-&gt;buffer [string-&gt;length] = c;
    if (c != '\0')
        string-&gt;buffer [++string-&gt;length] = '\0';
}</code></pre>

<pre><code class="language-c">#define vStringPut(s,c) \
    (void)(((s)-&gt;length + 1 == (s)-&gt;size ? vStringAutoResize (s) : 0), \
    ((s)-&gt;buffer [(s)-&gt;length] = (c)), \
    ((c) == '\0' ? 0 : ((s)-&gt;buffer [++(s)-&gt;length] = '\0')))
#endif</code></pre>

在string->length+1==string->size时，由于有末尾\0实际上存储已满，故增大buffer大小。

注意不能添加&#8217;\0&#8217;，否则结果会错误。

宏版本用,运算符精炼地连接了多个语句，用?来进行if的作用。

## void vStringCatS (vString _const string, const char _const s)

将s的内容增加到string的后面。

<pre><code class="language-c">void vStringCatS (vString *const string, const char *const s)
{
#if 1
    const size_t len = strlen (s);
    while (string-&gt;length + len + 1 &gt;= string-&gt;size)/*  check for buffer overflow */
        vStringAutoResize (string);
    strcpy (string-&gt;buffer + string-&gt;length, s);
    string-&gt;length += len;
#else
    const char *p = s;
    do
        vStringPut (string, *p);
    while (*p++ != '\0');
#endif
}</code></pre>

后面那一段是原本注释掉的内容，还是一句话：strcpy这些标准库内置的东西，在实现时都是经过反复测试的，在绝大多数情况下比自己写的要快要安全，不要重复造轮子。

为什么不用strcat？

因为在这里, string的结束地址是已知的（我们已经记录了length），strcat会从头扫一遍string，效率太低。

## void vStringNCatS (vString _const string, const char _const s, const size_t length)

将s的前length个字符拷贝到string中。

<pre><code class="language-c">void vStringNCatS (
        vString *const string, const char *const s, const size_t length)
{
    const char *p = s;
    size_t remain = length;

    while (*p != '\0'  &&  remain &gt; 0)
    {
        vStringPut (string, *p);
        --remain;
        ++p;
    }
    vStringTerminate (string); //在buffer末尾添加'\0'
}</code></pre>

这里为什么不使用strncpy？动态扩容固然是一方面，但如果我们一开始就扩容好呢？我们看一下strncpy的情况：

>   * If count is reached before the entire string src was copied, the resulting character array is not null-terminated. 
>   * If, after copying the terminating null character from src, count is not reached, additional null characters are written to dest until the total of count characters have been written.

简而言之：

  * 在length<s长度时，strncpy不会添加&#8217;\0&#8242; 
  * 在length==s长度时，strncpy会添加&#8217;\0&#8242; 
  * 在length>s长度时，strncpy会添加多个&#8217;\0&#8242;

而我们是要始终末尾有&#8217;\0&#8217;。虽然我们可以在末尾强制加一个&#8217;\0&#8217;快速解决问题，但在情况3时，效率低。