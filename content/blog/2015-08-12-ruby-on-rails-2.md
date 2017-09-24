---
title: Ruby on Rails (2)
author: htfy96
type: post
date: 2015-08-12T08:10:50+00:00
url: /2015/08/12/ruby-on-rails-2/
categories:
  - 代码

---
> 由于前一段时间太懒，直到今天才恢复学习。。。 

# Router转发

Router不仅仅会根据url来区分调用哪个函数……还会通过http请求的种类来划分。。。

           Prefix Verb   URI Pattern                  Controller#Action
    welcome_index GET    /welcome/index(.:format)     welcome#index
             root GET    /                            welcome#index
         articles GET    /articles(.:format)          articles#index
                  POST   /articles(.:format)          articles#create
      new_article GET    /articles/new(.:format)      articles#new
     edit_article GET    /articles/:id/edit(.:format) articles#edit
          article GET    /articles/:id(.:format)      articles#show
                  PATCH  /articles/:id(.:format)      articles#update
                  PUT    /articles/:id(.:format)      articles#update
                  DELETE /articles/:id(.:format)      articles#destroy

如图所见，`articles`的`index`和`create` url相同，但却调用了不同的函数

值得注意的是`patch`这种请求类型，是个新玩意http://tools.ietf.org/html/rfc5789 ，能只更新一部分内容

深入理解看这个： <https://ihower.tw/blog/archives/6483>

# Block

用

<pre><code class="language-ruby">&lt;% @article.each { |article| %&gt;
&lt;%= article.body %&gt;
&lt;% } %&gt;</code></pre>

之类的语法
  
好像经常出问题……换成`do |article| ... end`就好了，大概是因为中间的函数如果接受了一个`Hash`作为参数，本身的`{}`会发生一些奇怪的问题

# link_to

`<%= link_to 'My Blog', controller: 'articles' %>`
  
和
  
`<%= link_to 'My Blog', articles_path %>`好像是差不多的，不过在`articles`的外部用第一种居多，内部用第二种

`method: :delete, data: { confirm: 'Are you sure?' }`
  
method对应路由表的`VERB`

# Model的一些设置。。。

## 数据验证

<pre><code class="language-ruby">validates :title, presence: true,
                    length: { minimum: 5 }</code></pre>

其实是一个函数……不过没括号真心爽

## 关联

创建新表时就要

    rails generate model Comment commenter:string body:text article:references

> You can also consider `references` as a kind of type. For instance, if you run:
  
> `rails generate model photo title:string album:references`
  
> It will generate an `album_id` column. You should generate these kinds of fields when
  
> you will use a `belongs_to` association, for instance. `references` also supports
  
> polymorphism, you can enable polymorphism like this:
  
> `rails generate model product supplier:references` 

之后有几种配置：

  * belongs_to
  * has_one (through)
  * has_many (through)
  * has\_and\_belongs\_to\_many

<http://guides.ruby-china.org/association_basics.html>

    has_many :comments, dependent: :destroy

再之后如果是从属的话在路由表里可以搞：

    resouces :articles do
        :comments
    end

# 错误

      <% if @article.errors.any? %>
      <div id="error_explanation">
        <h2><%= pluralize(@article.errors.count, "error") %> prohibited
          this article from being saved:</h2>
        <ul>
        <% @article.errors.full_messages.each do |msg| %>
          <li><%= msg %></li>
        <% end %>
        </ul>
      </div>
      <% end %>

# Path

     new_article GET    /articles/new(.:format)      articles#new
     edit_article GET    /articles/:id/edit(.:format) articles#edit
     article GET    /articles/:id(.:format)      articles#show

查看路由表时我们可以看到`prefix`一列，那一列加上`_path`就是url的变量了。。

如`articles_path`, `edit_article_path(@article)`之类的

# 局部模板

文件名以`_`开头，调用`<% render 'template_name; %>`渲染，不需要开头下划线

# CURD简易速查。。。

    @article = Article.find(params[:id])
    if (@article.update(article_params))
    @articles = Article.all
    @article = Article.new(article_params)
    if @article.save
    @article.destroy

# 嵌套路由

    <h2>Add a comment:</h2>
    <%= form_for([@article, @article.comments.build]) do |f| %>
      <p>
        <%= f.label :commenter %><br>
        <%= f.text_field :commenter %>
      </p>
      <p>
        <%= f.label :body %><br>
        <%= f.text_area :body %>
      </p>
      <p>
        <%= f.submit %>
      </p>
    <% end %>