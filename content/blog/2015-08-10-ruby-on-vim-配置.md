---
title: Ruby on Vim 配置
author: htfy96
type: post
date: 2015-08-10T06:51:46+00:00
url: /2015/08/10/ruby-on-vim-配置/
categories:
  - 代码

---
使用**Neocomplete**作为补全框架，由于bug必须将补全权全部交给**ruby.vim**

<pre><code class="language-vim">if !exists('g:neocomplete#force_omni_input_patterns')
    let g:neocomplete#force_omni_input_patterns = {}
endif
let g:neocomplete#force_omni_input_patterns.ruby = '[^. *\t]\.\w*\|\h\w*::'
autocmd FileType ruby setlocal omnifunc=rubycomplete#Complete

let g:rubycomplete_classes_in_global = 1
let g:rubycomplete_rails = 1
let g:rubycomplete_load_gemfile = 1</code></pre>