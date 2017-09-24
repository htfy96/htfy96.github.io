---
title: Unofficial Syntax of Markdown
author: htfy96
type: post
date: 2015-02-25T05:01:13+00:00
url: /2015/02/25/unofficial-syntax-of-markdown/
categories:
  - 代码

---
# Tag

@(Maintag)[tag2|tag3|tag4]

# footnote

[Blah&#8230;][1]

* * *

[TOC]

# Code block

<pre><code class="language-python">@requires_authorization
def somefunc(param1='', param2=0):
    '''A docstring'''
    if param1 &gt; param2: # interesting
        print 'Greater'
    return (param2 - param1 + 1) or None
class SomeClass:
    pass
>&gt;&gt; message = '''interpreter
... prompt'''</code></pre>

# Latex

This is a formula: $\Gamma(n) = (n-1)!\quad\forall n\in\mathbb N$.

Another:

$$ x = \dfrac{-b \pm \sqrt{b^2 &#8211; 4ac}}{2a} $$

# Table

<table>
  <tr>
    <th style="text-align: left;">
      Item
    </th>
    
    <th style="text-align: right;">
      Value
    </th>
    
    <th style="text-align: center;">
      Qty
    </th>
  </tr>
  
  <tr>
    <td style="text-align: left;">
      Computer
    </td>
    
    <td style="text-align: right;">
      1600 USD
    </td>
    
    <td style="text-align: center;">
      5
    </td>
  </tr>
  
  <tr>
    <td style="text-align: left;">
      Phone
    </td>
    
    <td style="text-align: right;">
      12 USD
    </td>
    
    <td style="text-align: center;">
      12
    </td>
  </tr>
  
  <tr>
    <td style="text-align: left;">
      Pipe
    </td>
    
    <td style="text-align: right;">
      1 USD
    </td>
    
    <td style="text-align: center;">
      234
    </td>
  </tr>
</table>

# flowchart

<pre><code class="language-flow">st=&gt;start: Start
e=&gt;end
op=&gt;operation: My Operation
cond=&gt;condition: Yes or No?

st-&gt;op-&gt;cond
cond(yes)-&gt;e
cond(no)-&gt;op</code></pre>

# sequence chart

<pre><code class="language-sequence">Alice-&gt;Bob: Hello Bob, how are you?
Note right of Bob: Bob thinks
Bob--&gt;Alice: I am good thanks!</code></pre>

# checkbox

  * [x] Completed
  * [ ] item2
  * [ ] item3

 [1]: https://chrome.google.com/webstore/detail/kidnkfckhbdkfgbicccmdggmpgogehop