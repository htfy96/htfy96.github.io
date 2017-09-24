---
title: Network_algo设计及优化杂记
author: htfy96
type: post
date: 2016-02-19T04:59:09+00:00
url: /2016/02/19/network_algo设计及优化杂记/
categories:
  - 代码

---
## 问题来源

  * 原有的数据库太大，无法全部放在内存中跑起来算法。
  * 图专用数据库（Neo4j）、RDBMS，提供的功能太全导致速度太慢，同时Client-Server模型通讯成本高昂，所以选择嵌入式数据库。
  
    <img src="https://i1.wp.com/www.arangodb.com/wp-content/uploads/2015/06/chart_performance_r105-1024x576.jpg?resize=840%2C473&#038;ssl=1" alt="此处输入图片的描述" data-recalc-dims="1" />
  * 引用图深度浅，数目多，所以选用Key-Value作为存储
  
    <img src="https://i1.wp.com/dev.assets.neo4j.com.s3.amazonaws.com/wp-content/uploads/nosql-space.png?w=840&#038;ssl=1" alt="存在关系型数据库中（MySQL）接口不统一" data-recalc-dims="1" />
  * 数据结构难以与算法分离

## 目标

通用的单机图论算法框架。

  * 通用性
  * 高性能

## 概述

### 使用步骤

#### 安装

    git clone --recursive https://github.com/htfy96/network_algo.git
    sudo add-apt-repository ppa:ubuntu-toolchain-r/test -y
    sudo apt-get -qq update
    sudo apt-get install -y libmysqlclient-dev cmake libsqlite3-dev libleveldb-dev libprotobuf-dev protobuf-compiler libsnappy-dev
    sudo apt-get install gcc-5 g++-5 -y //GCC 5+必需
    export CXX="g++-5" CC="gcc-5" //编译时必需
    
    cd network_algo
    mkdir build && cd build
    cmake ..
    make -j2
    sudo make install

#### 定义图的节点、边

<pre><code class="language-proto">message Node
{
    required string id = 1; //必须是这个类型和名称
    required double dummy = 2; //自定义数据
    //...
}
message Edge
{
    required string id = 1; //必须
    required string from = 2; //必须
    required string to = 3; //必须
    required int32 dummy = 4; //自定义数据
    //...
}</code></pre>

保存成`graphproto.proto`，然后`protoc graphproto.proto --cpp_out=你自身代码所在的目录`，会生成`graphproto.pb.h`和`graphproto.pb.cc`。
  
（`protoc`是`protobuf-compiler`包内的程序）

#### 写代码

<pre><code class="language-cpp">#include "graphproto.pb.h" //里面包含了自动生成的Node和Edge
#include &lt;netalgo/include/leveldbgraph.hpp&gt;
#include &lt;netalgo/include/graphdsl.hpp&gt;
// N --&gt; M &lt;-- K
using namespace netalgo;
using namespace std;
int main()
{
    LeveldbGraph&lt;Node, Edge&gt; graph("mygraph.db");
    Node n; n.set_id("N"); n.set_dummy(2.33);
    Node m; m.set_id("M"); m.set_dummy(6.66);
    Node k; k.set_id("K"); k.set_dummy(-2.5);
    Edge nm; nm.set_id("NtoM"); nm.set_from("N"); nm.set_to("M"); nm.set_dummy(1);
    Edge km; km.set_id("KtoM"); km.set_from("K"); km.set_to("M"); km.set_dummy(2);
    graph.setNode(n); graph.setNode(m); graph.setNode(k); //setEdge can be used in a similar way
    vector&lt;Edge&gt; edgeVec; edgeVec.push_back(nm); edgeVec.push_back(km);
    graph.setEdgesBundle(edgeVec); //setNodesBundle

    auto start = graph.parse("match (a)-[e1]-&gt;(mid id="M")&lt;-[e2]-(c) return a,e1,c"_graphsql); // return an iterator
    for (auto it=start; it!=graph.end(); ++it) //iterates over result set
    {
        cout &lt;&lt; it-&gt;getNode("a").id() &lt;&lt; " " &lt;&lt; it-&gt;getNode("c").id() &lt;&lt; endl; //N K
        cout &lt;&lt; it-&gt;getEdge("e1").dummy() &lt;&lt; endl; //1
    }

    cout &lt;&lt; calcDensity(graph) &lt;&lt; endl; //use existing algorithm
}</code></pre>

编译：

`g++-5 -std=c++11 your_prog.cpp graphproto.cc -lleveldb -lsnappy -lprotobuf -lnetalgo -o your_prog`

  * 必须使用`g++ 5.0`及以上版本编译，否则会报错，可以用`gcc -v`来查看`gcc`的版本
  * `graphproto.cc`是`graphproto.proto`编译出来的东西
  * `-lleveldb -lsnappy`是`LevelDb`这个数据库的运行库，只有在使用`LeveldbGraph`类的时候才会用到
  * `-lprotobuf`是为了支持`graphproto.pb.h`所需要的库，必需
  * `-lnetalgo`是这个算法库的东西

## 架构

<img src="https://i0.wp.com/ooo.0o0.ooo/2016/02/18/56c6a1515f7ed.png?w=840&#038;ssl=1" alt="此处输入图片的描述" data-recalc-dims="1" />

构想是将整个库分成3层：

  * **算法层** 通过graphsql向协议层发送消息，并通过协议层返回的迭代器来获取数据。与`GraphType`是什么没有任何关系。本层可以由用户自己写，库自身也会提供一些算法。 
    <pre><code class="language-cpp">template&lt;typename GraphType&gt;
double calcSomething(GraphType& graph)
{
auto querySentence = "match (a)--(b)--(c) return a,c"_graphsql; //发送的消息，利用协议层提供的`_graphsql`后缀解析完成
auto start = graph.query(querySentence); //向graph发送查询请求，返回迭代器
for (auto it = start; it!=graph.end(); ++it)
    cout &lt;&lt; it-&gt;getNode("a").id() &lt;&lt; it-&gt;getNode("c").id() &lt;&lt; endl; //通过迭代器来获取实际信息
}</code></pre>

  * **协议层** 本层全部由库提供。 主要包括`_graphsql`这个后缀（将查询字符串转化为`GraphSqlSentence`）和各个Graph实现都要继承的`GraphInterface`基类，约束了各个实现所需要的接口（如`set_node`方法）。同时还给数据库层提供了一些协助函数（如`compareProperty(data, propertyName, opCode, value)`）。 
  * **数据库层** 封装了一个图存储的数据结构，需要满足协议层`GraphInterface`基类的接口。 本库将提供`LeveldbGraph`这个结构，用户也可以参考其实现自己的数据结构。

## 具体实现

### _graphsql

实现了一个字符串到`GraphSqlSentence`类的解释器。

  * 对`const char* / const string`缓存它们的parse结果。</p> 
    <pre><code class="language-cpp">for (int i=0; i&lt;100000; ++i) {
auto sentence = 
"match (a)--(b) return a,b"_graphsql; //在这100000次循环中只有第一次会真正解析一遍这个字符串，其余时候都会缓存下来
auto mutableSentence =
graphSqlParse(string("match (a)--(b id=\"") + string(1, 'a'+(i%26)) +"\") return a")); //这种情况每次都必须要解析一遍
}</code></pre>
    
    ### GraphInterface
    
    图的接口。
    
    <pre><code class="language-cpp">template&lt;typename NodeType, EdgeType&gt;
class GraphInterface
{
//...
public:
    virtual ResultType query(const GraphSqlSentence&) = 0;
    virtual void setNode(const NodeType&) = 0;
    virtual void setEdge(const EdgeType&) = 0;
    virtual void setNodesBundle(const NodesBundle&) = 0;
    virtual void setEdgesBundle(const EdgesBundle&) = 0;

    virtual void removeNode(const NodeIdType&) = 0;
    virtual void removeEdge(const EdgeIdType&) = 0;

    virtual void destroy() = 0;
};</code></pre>

### LeveldbGraph

基于LevelDb的高性能图数据库。维护`<NodeId:@outEdges =>  set<EdgeIdT>>`, `<NodeId:@inEdges => set<EdgeIdT>>`, `<NodeId:@nodeData => Node>`, `<EdgeId:@edgeData => Edge>`这四组键值对来实现存储。

  * 利用`Protobuf`和`cereal`库，分别对`Node`和`Edge`（它们都由`protoc`编译生成）， `set<EdgeIdT>`进行高效的序列化及反序列化
  * 实现了`TypedLastUsedMap`，并用其缓存`inEdges`和`outEdges`，在大量密集读取的时候可以实现接近内存速度。
  * `query`的匹配算法：
  
    `match (a)<-[b id="xxx"]-(c)-[e]->(d id="yyy") return a,b,c,d,e`
  
    首先，`b`边和`d`点可以直接确定，以它们为中心把链切割成2段：</p> 
      1. `(a)<-[b id="xxx"]`，已知`b`边后可直接知道`b`的`to()`就是`a`点的`id()`，直接获取
      2. `[b id="xxx"]-(c)-[e]->(d id="yyy")`，此链两端点已知，双向匹配： 
          1. `b.from()` == `c.id()`
          2. 获取`d.inEdges()`，得到`d`所有入边的`EdgeId`
          3. 获取`c.outEdges()`，得到`c`所有出边的`EdgeId`
          4. 把`2`，`3`得到的`edgeId`取交集，所得唯一结果就是`e.id()`