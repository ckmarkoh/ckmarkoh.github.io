---
layout: post
title: 'Python Functional Programming Style 1'
date: 2014-03-18 07:04
comments: true
categories: ['Python Programming','Functional Programming']
---

今天來看看如何用python的Functional Programming tool來簡化程式碼

所謂的Functional Programming, 是指像是Lisp或Haskell這樣的程式語言

python所提供的Functional Programming tool

可以用Functional Programming這些語言的風格來寫python

<!--more-->

## 1. lambda

lambda這個詞是來自於Lambda Calculus

簡而言之, Lambda Calculus是一種計算理論

若想了解Lambda Calculus是什麼, 請看:http://en.wikipedia.org/wiki/Lambda_calculus

而 *lambda* 這個key word在python裡面是用來定義anonymous functions或small function

例如：


```python

>>> my_function = lambda x : x+2
>>> print my_function(1)
3

```


或：


```python

>>> print (lambda x : x+2 ) (1)
3

```


## 2. map


假設有一個function和一個list,如下：


```python

>>> my_function = lambda x : x+3
>>> my_list = [1,2,3,4,5]

```


如果要把 `my_list` 中的每個element用 `my_function` 做運算

例如：


```python

>>> my_list_2 = []
>>> for x in my_list:
...     my_list_2.append(my_function(x))

>>> print my_list_2
[4, 5, 6, 7, 8]

```


這可以用map改成這樣：


```python

>>> my_list_2 = map(my_function,my_list)
>>> print my_list_2
[4, 5, 6, 7, 8]

```


也可以直接這樣寫



```python

>>> my_list_2 = map(lambda x : x+3,[1,2,3,4,5])
>>> print my_list_2
[4, 5, 6, 7, 8]

```


這樣都可以把for迴圈化簡掉


map也可以用於多個argument的function

例如：



```python

>>> print map(lambda x,y : x*y ,[1,2,3,4], [2,4,6,8])
[2, 8, 18, 32]

```


## 3.filter


給定一個list如下



```python

>>> my_list = [1,2,3,4,5,6,7,8]

```


假如要取出 `my_list` 中為偶數的數字,如下：



```python

>>> my_list_2 = []
>>> for x in my_list:
...     if x%2 == 0:
...             my_list_2.append(x)

>>> print my_list_2
[2, 4, 6, 8]

```


filter的概念就是去掉function運算後為False的element,如下：



```python

>>> my_list_2 = filter(lambda x : x%2 == 0, my_list)
>>> print my_list_2
[2, 4, 6, 8]

```


這樣就可以省略掉for和if,把成式碼合併到一行之中


再一例子,如果要把 `my_list` 中偶數的element取出來後平方,可以這樣寫：



```python

>>> my_list_2 = filter(lambda x : x**2 if x%2==0 else False, my_list)
>>> print my_list_2
[2, 4, 6, 8]

```


## 4.reduce


給定一個list如下



```python

>>> my_list = [3,2,1,6,5,4]

```


如果要算出這個例子的總和,用for迴圈會這樣寫



```python

>>> sum=0
>>> for x in my_list:
...     sum+=x

>>> print sum
21

```


這樣至少要寫三行,如果用reduce的話一行就夠了



```python

>>> sum = reduce(lambda a,b:a+b,my_list)
>>> print sum
21

```


reduce的概念如下,用這個function把list中前兩個合併成一個

一直這樣合併下去


以下是reduce的分解過程

把程式碼複製後貼到 *my_reduce.py* 檔案



```python my_reduce.py
def my_reduce(my_fun,my_list):
      print my_list
      if len(my_list) == 1:
              return my_list[0]
      elif len(my_list) >= 2:
              my_list_1 = my_fun(my_list[0], my_list[1])
              return my_reduce(my_fun, [my_list_1] + my_list[2:])
      else:
      		    print "Error: reduce of empty sequence"
              return False

```


到interactive mode 執行看看



```python

>>> from my_reduce import *
>>> my_reduce(lambda a,b:a+b,my_list)
[3, 2, 1, 6, 5, 4]
[5, 1, 6, 5, 4]
[6, 6, 5, 4]
[12, 5, 4]
[17, 4]
[21]
21

```


看到了list前面兩個被function合併成一個,這樣繼續合併下去,得出結果

再舉一例,

如果要求出my_list中的最大值, 可以這樣：



```python

>>> max = reduce(lambda a,b: a if a>=b else b, my_list)
>>> print max
6

```


分解過程如下：



```python

>>> my_reduce(lambda a,b: a if a>=b else b,my_list)
[3, 2, 1, 6, 5, 4]
[3, 1, 6, 5, 4]
[3, 6, 5, 4]
[6, 5, 4]
[6, 4]
[6]
6

```


## 5.Further reading:


想要看更多關於lambda,map,filter和reduce的用法,請到：

http://www.python-course.eu/lambda.php

http://docs.python.org/2/tutorial/datastructures.html#functional-programming-tools
