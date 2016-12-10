---
layout: post
title: 'Python Standard Library -- urlparse'
date: 2014-03-11 02:56
comments: true
categories: [Python, web_crawler, python_standard_library]
---

今天來談到URL parse(URL剖析)


在製作web crawler(爬網頁程式)的時候

常常會需要用到URL剖析


## 1.URL的組成架構

URL的組成架構如下：

**[scheme]:/[net_loc]/[path];[params]?[query]#[fragment]**


舉個例子

**http://www.espn.com:80/basketball/nba/index.html;lang=engl?team=dallas#Roster**


則此URL的架構為：

[scheme]      http

[net_loc]     www.espn.com:80

[path]        /basketball/nba/index.html

[params]      lang=engl

[query]       team=dallas

[fragment]    Roster

<!--more-->

## 2.python urlparse

接著我們來看看如何用python來剖析URL


首先 載入urlparse模組


```python

>>> import urlparse 

```

再來我們看看如何做URL剖析



```python

>>> result=urlparse.urlparse("http://www.espn.com:80/basketball/nba/index.html;lang=engl?team=dallas#Roster")
>>> print result
ParseResult(scheme='http', netloc='www.espn.com:80', path='/basketball/nba/index.html', params='lang=engl', query='team=dallas', fragment='Roster')

```


取出結果中各個成分的方法,如下：


```python

>>> result.path
'/basketball/nba/index.html'

>>> result.query
'team=dallas'

```


## 3.python urljoin

urljoin是當我們要從一個網頁要跳到另外一個網頁時,會需要用到的

例如要從這個網頁

http://blog.storysensecomputing.com/whatsthenumber-zh/

跳到另外一個網頁

http://blog.storysensecomputing.com/info/

但是在html碼的超連結是這樣的,

`<a href="/info/">我們公司 / Our Company</a>`


如何從html碼中得出正確網址

就要用到urljoin了,用法如下：


```python

>>> urlparse.urljoin("http://blog.storysensecomputing.com/whatsthenumber-zh/","/info/")
'http://blog.storysensecomputing.com/info/'

```


還有其他各種變化形的url位置,沒關係,urljoin都可以處理,例如：


```python

>>> urlparse.urljoin("http://www.abc.com/aaa/bbb/ccc","/ddd/")
'http://www.abc.com/ddd/'

>>> urlparse.urljoin("http://www.abc.com/aaa/bbb/ccc","../ddd/")
'http://www.abc.com/aaa/ddd/'

>>> urlparse.urljoin("http://www.abc.com/aaa/bbb/ccc","../../ddd/")
'http://www.abc.com/ddd/'

>>> urlparse.urljoin("http://www.abc.com/aaa/bbb/ccc","//www.ddd.com/")
'http://www.ddd.com/'

```


## 結語：

以上是urlparse很簡單的介紹

想要更深入了解

關於urlparse的用法,可參考：http://docs.python.org/2/library/urlparse.html

網頁爬蟲的實作,可參考：http://faculty.cs.byu.edu/~rodham/cs240/crawler/

或：http://staypython.blogspot.tw/2012/06/python_25.html