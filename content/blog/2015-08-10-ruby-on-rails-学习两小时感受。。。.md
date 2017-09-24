---
title: Ruby on rails 学习两小时感受。。。
author: htfy96
type: post
date: 2015-08-10T08:19:28+00:00
url: /2015/08/10/ruby-on-rails-学习两小时感受。。。/
categories:
  - 代码

---
# Route

`config/routes.rb`

最开始的部件

决定了如何分发你的URL请求，把url一部分解析成变量丢给后面的东西：

  * `root 'welcome#index'` 主页
  * `get 'welcome#index'` 简单视图
  * `get 'products/:id' => 'catalog#view'`简单解析个id变量然后直接丢给视图
  * `resources :products` 把`products/`丢给products这个controller处理

# Controller

<pre><code class="language-ruby">class ArticlesController &lt; ApplicationController
    def new
    end
    def create
        @article = Article.new(article_params)
        @article.save
        redirect_to @article
    end

    def show
        @article = Article.find(params[:id])
    end

    private
    def article_params
        params.require(:article).permit(:title, :text)
    end
end</code></pre>

下面定义的各个成员函数就对应着`articles/***`的不同路径，派发给`views/articles/***.html.erb`的视图

视图可以访问controller本身的成员，因此应该先把路由分配出的`params[变量名]`解析成具体的程序数据（通过和Model操作）

因为权限原因，要显式说明要用哪些路由中的参数

# View

把controller中的数据转化成网页

有很多奇怪的函数解决：

<pre><code class="language-html">&lt;h1&gt;
    new Article
&lt;/h1&gt;

&lt;%= form_for :article, url: articles_path do |f| %&gt;
    &lt;p&gt;
        &lt;%= f.label :title %&gt; &lt;br /&gt;
        &lt;%= f.text_field :title %&gt;
    &lt;/p&gt;

    &lt;p&gt;
        &lt;%= f.label :text %&gt; &lt;br /&gt;
        &lt;%= f.text_area :text %&gt; 
    &lt;/p&gt;
    &lt;p&gt;
        &lt;%= f.submit %&gt;
    &lt;/p&gt;
&lt;% end %&gt;</code></pre>

# Model

Model规定了一类数据有哪些成员，应如何存储。

一个Model其实就是一张表。

    :Rails g model Article title:string text:text

然后要migrate创建这张表

如上所述，model支持find, new等操作。。。

To be continued &#8230;

> _Symbols_ <http://www.ibm.com/developerworks/cn/opensource/os-cn-rubysbl/> 明天读