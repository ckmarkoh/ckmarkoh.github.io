---
layout: post
title: 'Python -- List Comprehension'
date: 2014-02-16 04:13
comments: true
categories: [Python, list_comprehension]
---

List comprehension(列表示推導)是一種可以讓程式碼更簡潔，增加可讀性與執行效率的方法。

可以將很多行的for迴圈縮在短短一行之內，很方便！


## 1.

假如我們現在有一個list--l1


```python
l1=[2,5,3,8,4,9]

```

若我們想要建立一個新表單l2,是把l1中的每一個元素加上二,最先想到的寫法是：


```python
l2=[]
for l in l1: 
   l2.append(l+2) 

```

知道怎麼用List comprehension的話，我們可以用列表示: 


```python
l2=[l+2 for l in l1] 

```

這樣看起來就簡潔多了！

<!--more-->

## 2.

我們也可以把if條件判斷也寫進去。

例如，想要把l1中可以被3整除的數抓出來,存到l2


```python
l2=[]
for l in l1: 
   if l%3==0:
      l2.append(l)

```

用List comprehension來簡化，可以變成：


```python
l2=[l for l in l1 if l%3==0]

```


## 3.

超過一個以上的list也可以使用List comprehension，例如，兩個list


```python
x=[1,2,3]
y=[3,5,6]

```

如果我們要產生一個新的list--z是以每個x和每個y的元素的值為座標。


```python
z=[]
for xi in x:
   for yi in y:
      z.append((xi,yi))

```

這可以簡化成：


```python
z=[(xi,yi) for xi in x for yi in y] 

```


## 4.

其實List comprehension不只可以用在list,也可以用在dictionary上。

有一個list--z1如下。


```python
z1=[(1,3),(2,5),(3,6)]

```

例如我們想把z1改成dictionary--z2，像這樣：


```python
z2={}
for zi,zj in z1:
   z2.update({zi:zj})

```

可以簡化成：


```python
z2={zi:zj for zi,zj in z1} 

```


## 結語：

總之List comprehension可以大幅減少程式碼的行數。

其實，還有其他種方法，也可以讓程式碼更簡潔，例如map,filter和reduce，下一回我會提到。


想要看看更多有關於List comprehension可以到：

http://docs.python.org/2/tutorial/datastructures.html#list-comprehensions




