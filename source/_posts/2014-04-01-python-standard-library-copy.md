---
layout: post
title: 'Python Standard Library -- copy'
date: 2014-04-01 16:23
comments: true
categories: [python_standard_library]
---


## 1.Introduction


在python裡面, `=` 這個符號,

有可能是 *pass by value* 或是 *pass by reference*

如果 `=` 右方的 *variable* 是 *value* , 例如 *int*

則 `=` 是 *pass by value* ,如下



```python

>>> x1=1
>>> x2=x1
>>> x1+=1
>>> print x1
2

>>> print x2
1

```


![p1](/images/pic/pic_00001.tiff)

<!--more-->


我們讓 `x2=x1` , 但 `x1` 的值變了, `x1` 的值不變


但如果  `=` 右方的 *variable* 是 *object* , 例如 *list*

這個時候 `=` 就只是把 *pointer* 複製而已



```python

>>> y1=[]
>>> y2=y1
>>> print y2
[]

```


![p2](/images/pic/pic_00002.tiff)


如上圖, `y2` 和 `y1` 共同指向一個 *list* , 




```

>>> y1+=[1]
>>> print y1
[1]

>>> print y2
[1]

```


![p3](/images/pic/pic_00003.tiff)



如果 `y1` 改變, `y2` 也一起改變


## 2.copy.copy


先載入 `copy` 模組



```python

>>> import copy

```


如果要把 *variable* 所指到的 *object* 一起複製, 

而不是只複製 *pointer* , 就要用到 `copy.copy`




```python

>>> y1=[]
>>> y2=copy.copy(y1)
>>> print y2
[]

```


![p4](/images/pic/pic_00004.tiff)


如上圖, `y1` 所指的 *list* 也被複製了



```python

>>> y1+=[1]
>>> print y1
[1]

>>> print y2
[]

```


![p5](/images/pic/pic_00005.tiff)


所以當 `y1` 改變的時候, `y2` 不變



但是如果是多層的 *list* ,

使用 `copy.copy` 就可能會出問題了

因為 `copy.copy` 只會複製一層的 *object*




```python

>>> y0=[]
>>> y1=[y0]
>>> y2=copy.copy(y1)
>>> print y2
[[]]

```


![p6](/images/pic/pic_00006.tiff)


複製超過一層以後,剩下的就只複製 *pointer*, 如上圖：




```python

>>> y0+=[1]
>>> print y0
[1]

>>> print y1
[[1]]

>>> print y2
[[1]]

```


![p7](/images/pic/pic_00007.tiff)


這個時候 `y0` 改變了, `y1` 和 `y2` 都會變



## 3.copy.deepcopy


如果要把每一層所有的 *object* 都複製,

而不要複製 *pointer*

就要用 `copy.deepcopy`



```python

>>> y0=[]
>>> y1=[y0]
>>> y2=copy.deepcopy(y1)
>>> print y2
[[]]

```


![p8](/images/pic/pic_00008.tiff)


如上圖, `copy.deepcopy` 會把每層的 *object* 都複製



```python

>>> y0+=[1]
>>> print y0
[1]

>>> print y1
[[1]]

>>> print y2
[[]]

```


![p9](/images/pic/pic_00009.tiff)


這個時候若 `y0` 改變了, 只有 `y1`會變 , 而 `y2` 不會變



## 4.Reference

https://docs.python.org/2/library/copy.html