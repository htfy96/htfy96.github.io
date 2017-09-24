---
title: SJTUACM OJ 1246 Pyramid
author: htfy96
type: post
date: 2016-04-23T21:45:36+00:00
url: /2016/04/24/sjtuacm-oj-1246-pyramid/
categories:
  - 代码

---
## 题目

<http://acm.sjtu.edu.cn/OnlineJudge/problem/1246>

### Description

Jaguar 国王在一场战役大胜后决定建造一个金字塔，一方面作为纪念战争胜利的纪念碑，同时亦用作埋葬在战斗中阵亡将士们的墓地。该金字塔将建在战场的所在地，有一个a列b行的矩形底部。在金字塔的底层内将有一个较小的c列d行的矩形墓室，用来存放阵亡将士们的遗体以及他们所用过的武器。

国王的建筑师将该战场分为m列n行小方格的网格。每个小方格的高度用一个整数表示。

金字塔及其内部的墓室均应覆盖网格的完整的小方格，而且其边必须与战场的边平行。

金字塔内墓室小方格的高度必须保持原有的高度不变，但金字塔底部的其余地面则需要平整，平整的方法是将高度较高的小方格内的砂土移到较低的小方格内，这样一来，金字塔底部的最后高度将是底部所覆盖的所有小方格（墓室小方格除外）高度的平均值。建筑师有权选择墓室在金字塔内的位置，但墓室四周要有至少一小方格厚度的墙围绕。

建筑师希望在战场上找到一块最佳的位置来建造该金字塔及其内部的墓室，使得对于给定大小的金字塔，其底部的高度尽可能高。
  
<img src="https://i0.wp.com/acm.sjtu.edu.cn/ds/_media/pyramid.gif?w=840" alt=" " title="graph" data-recalc-dims="1" />
  
图中所示是一个战场的例子，每个小方格内的整数表示该格所处区域内的高度。灰色部份表示金字塔的底部，灰色部份所包围的白色部份则表示墓室的位置。该图给出了一个最佳位置。

对于给定尺寸的战场、金字塔和墓室大小以及战场上所有小方格的高度，请编写一程序找出建造金字塔及其墓室的最佳位置，使得金字塔底部的高度为最高。 数据保证存在唯一解。

### Input Format

第一行：共有6个以空格分开的正整数，它们顺次分别为：m, n, a, b, c 及 d。

随后n行：每行有m个用空格分开的整数，每行表示一横行小方格的高度。其中，第一行表示战场小方格最上面的一行（第一行），而最后一行表示最下面的一行（第n行）。每行的m个整数依次表示该行上从第一列开始的小方格的高度。

70%: 3 <= n, m <= 10;

80%: 3 <= n, m <= 100;

100%: 3<= n, m <= 1000; 3 <= a <= m, 3 <= b <= n; 1 <= c, d <= a &#8211; 2, b &#8211; 2; 1 <= h <= 100

### Output Format

第一行：必须输出两个由空格分隔的整数，它们表示金字塔的底部的左上角位置，其中第一个整数代表列的坐标，而第二个整数则代表行的坐标。

第二行：必须输出两个由空格分隔的整数，它们表示金字塔内部的墓室的左上角位置，其中第一个整数表示列的坐标，而第二个数字则表示行的坐标。

### Sample Input

    8 5 5 3 2 1
    1 5 10 3 7 1 2 5
    6 12 4 4 3 3 1 5
    2 4 3 1 6 6 19 8
    1 1 1 3 4 2 4 5
    6 6 3 3 3 2 2 2

### Sample Output

    4 1
    6 2

## 解

好老的题目……总感觉在哪里见过但是想不起来了……

其实就是先遍历所有的a x b矩形(和为`S1`)，然后在每个中找和(`S2`)最小的c x d矩形。最后求`Max { S1 - S2 }`

### 首先注意到我们可以在O(mn)时间内求出所有的定大小矩形内数的和……

    S[i][j] = S[i-1][j] + S[i][j-1] + S[i-1][j-1] - A[i-h][j] - A[i][j-w] + A[i-h][j-w] + A[i][j]

`S[i][j]`代表以`(i,j)`为右下角，大小为w x h矩形内数的和。它可以由左上角的三个矩形加加减减算出来。

边界情况很好处理，只要认为所有出界的A都是0就好了。

但是因为之后的原因，对于`i<h || j<w`的情况，我们需要在最后全部填成+Inf.

<pre><code class="language-cpp">inline int C(int x, int y)
{
    if (x&gt;0 && y&gt;0) return a[x][y]; else return 0;
}

void calcSum(int w, int h, int (*arr)[1002])
{
    for (int i=1; i&lt;=m; ++i)
        for (int j=1; j&lt;=n; ++j)
        {
                arr[i][j] = arr[i-1][j] + arr[i][j-1] - arr[i-1][j-1] -C(i-h, j) -C(i, j-w) + C(i-h, j-w)+ a[i][j];
        }
    for (int i=1; i&lt;=m; ++i)
        for (int j=1; j&lt;=n; ++j)
        {
            if (i&lt;w || j&lt;h) arr[i][j] = 10000000;
            if (i&gt;=w || j&gt;=h) break;
        }
}</code></pre>

### 接下来扫一遍整个数组

`S1`已经计算出来了，接下来就是怎么找出a-c-1 x b-d-1矩形中`S2`的最小值了……

有好多好多方法可以搞！

#### ST表

自己查吧……空间/时间复杂度O(mnlogm logn)

#### Map

开个std::map, 然后框向右移的时候把左边那一列的每个元素删掉，再加入右边的每个元素
  
空间复杂度O(mn) 每一列的点共需要进/出 `(n-h)h`次，当`h==n/2`时达到最大值，所以是O(mn^2logn)次，这样<del>居然</del>可以卡过去……（<del>数据太弱了。。。</del>）

#### 预处理出某一小列，然后堆

#### 优先队列

空间复杂度O(mn) 时间复杂度O(mn)

优先队列的基础知识这里就不说了，主要就是能在O(l)时间内维护一个l长数组内，任意连续k个数字中的最值。

拓展到二维其实是一样的……先预处理出竖着h个元素的最小值(O(mn))，然后在横着求w个最小值的最小值(O(mn))，这样就能维护起来了

最后记得开读入优化……

## Code

<pre><code class="language-cpp">using namespace std;
int m,n,aa,bb,cc,dd;

int a[1002][1002], s1[1002][1002], s2[1002][1002];

inline int C(int x, int y)
{
    if (x&gt;0 && y&gt;0) return a[x][y]; else return 0;
}

void calcSum(int w, int h, int (*arr)[1002])
{
    for (int i=1; i&lt;=m; ++i)
        for (int j=1; j&lt;=n; ++j)
        {
                arr[i][j] = arr[i-1][j] + arr[i][j-1] - arr[i-1][j-1] -C(i-h, j) -C(i, j-w) + C(i-h, j-w)+ a[i][j];
        }
    for (int i=1; i&lt;=m; ++i)
        for (int j=1; j&lt;=n; ++j)
        {
            if (i&lt;w || j&lt;h) arr[i][j] = 10000000;
            if (i&gt;=w || j&gt;=h) break;
        }
}

deque&lt; pair&lt;int, int&gt; &gt; q; //first = pos  second=val

int mins2[1002][1002], minmins2[1002][1002];
int mins2x[1002][1002], mins2y[1002][1002];

int main()
{
    nosync;
    cin &gt;&gt; n &gt;&gt; m &gt;&gt; bb &gt;&gt; aa &gt;&gt; dd &gt;&gt; cc;
    for (int i=1; i&lt;=m; ++i)
        for (int j=1; j&lt;=n; ++j)
            cin &gt;&gt; a[i][j];

    calcSum(bb, aa, s1);
    calcSum(dd, cc, s2);

    for (int j=1; j&lt;=n; ++j)
    {
        q.clear();
        for (int i=1; i&lt;=m; ++i)
        {
            while(!q.empty() && q.front().first &lt;= i-(aa-cc-1)) q.pop_front();
            while (!q.empty() && q.back().second &gt;= s2[i][j]) q.pop_back();
            q.push_back(make_pair(i, s2[i][j]));
            mins2[i][j]=q.front().second; //竖着h个元素中的最小值
            mins2x[i][j] = q.front().first; //最小值的行
        }
    }

    for (int i=1; i&lt;=m; ++i)
    {
        q.clear();
        for (int j=1; j&lt;=n; ++j)
        {
            while (!q.empty() && q.front().first &lt;= j-(bb-dd-1)) q.pop_front();
            while (!q.empty() && q.back().second &gt;= mins2[i][j]) q.pop_back();
            q.push_back(make_pair(j, mins2[i][j]));
            minmins2[i][j] = q.front().second; //横着w个元素最小值
            mins2y[i][j]=q.front().first; //最小值所在的列
        }
    }
    long long ma = -1000000;
    int minx, miny, minxx, minyy;
    for (int i=1; i&lt;=m; ++i)
    {
        for (int j=1; j&lt;=n; ++j)
        {
            if (i&lt;aa || j&lt;bb) continue;
            if (s1[i][j] - minmins2[i-1][j-1] &gt; ma)
            {
                ma = s1[i][j] - minmins2[i-1][j-1]; //最小值已经跑出来了
                minx = i; miny = j;
                minyy = mins2y[i-1][j-1];
                minxx = mins2x[i-1][minyy]; //获得所在的行列
            }
        }
    }
    cout &lt;&lt; miny - bb + 1 &lt;&lt; " " &lt;&lt; minx - aa + 1 &lt;&lt; endl;
    cout &lt;&lt; minyy - dd + 1 &lt;&lt;" "&lt;&lt;minxx - cc + 1   &lt;&lt;endl;
}</code></pre>