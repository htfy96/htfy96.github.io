---
title: 扇贝批量加词（Shell script）
author: htfy96
type: post
date: 2017-05-10T11:43:35+00:00
url: /2017/05/10/扇贝批量加词（shell-script）/
categories:
  - 代码
tags:
  - shell
  - shell script
  - 扇贝英语

---
先每行一个写入words.txt，然后网页端加一个，开发者工具 &#8211; 网络 &#8211; Copy as curl，得到如下形式：

<pre>curl 'https://www.shanbay.com/bdc/vocabulary/add/batch/?words=clerical&_=1494415650964' -H 'Host: www.shanbay.com' -H 'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:53.0) Gecko/20100101 Firefox/53.0' -H 'Accept: application/json, text/javascript, */*; q=0.01' -H 'Accept-Language: zh-CN,en-US;q=0.7,en;q=0.3' --compressed -H 'X-Requested-With: XMLHttpRequest' -H 'Referer: https://www.shanbay.com/bdc/vocabulary/add/batch/' -H 'Cookie: csrftoken=?????; _ga=GA?????; _gat=1; auth_token=????????; sessionid="??????"; userid=?????; language_code=zh-CN' -H 'DNT: 1' -H 'Connection: keep-alive' -H 'Pragma: no-cache' -H 'Cache-Control: no-cache'</pre>

然后新建一个`add_single.sh`，写入

<pre>#!/bin/bash
curl "https://www.shanbay.com/bdc/vocabulary/add/batch/?words=${1}&_=1494415650964" ..............</pre>

之后`chmod +x add_single.sh`

最后安装parallel工具，

<pre>cat words.txt | parallel ./add_single.sh</pre>