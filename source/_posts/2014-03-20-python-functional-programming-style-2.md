---
layout: post
title: 'Python -- Functional Programming Style 2  '
date: 2014-03-20 13:35
comments: true
categories: [functional_programming, Python, operator, functools]
---

用Functional Programming style來寫程式的時候

可以大幅減少行數和變數的數量

本文接續上一篇:/blog/2014/03/18/python-functional-programming-style-1

繼續探討這種Functional Programming style在python中的應用


## 1. operator


首先,載入 `operator` 這個模組



```python

>>> import operator

```


然後,來介紹到底要怎麼使用

<!--more-->

#### 1.1, operator.add, operator.multiply, etc


`operator` 是一個提供加減乘除之類運算的module,

有的時候不太方便直接用到 `+` , `-` 這些符號,

這時候就要用到 `operator` 這個模組了


如果要求一個list中所有元素的總和,如果不會functional programming style

最基本的寫法,用for迴圈寫,如下



```python

>>> s=0
>>> for i in [3,5,7,9,11]:
...     s += i 

>>> print s
35

```


這樣要寫很多行

可以用lambda function簡化


```python 

>>> reduce(lambda a,b:a+b,[3,5,7,9,11])
35

```


但其實可以連lambda function都不用寫

因為python就已經有內建好這些function


我們可以用 `operator.add` 搭配 `reduce` 就可以了



```python 

>>> reduce(operator.add,[3,5,7,9,11])
35

```


這就是 `operator` 的用法,可以把＋,-,之類的operator運算當成function來使用

事實上,根本什麼都不用寫,用內建的function `sum()` 就可以了



```python

>>> sum([3,5,7,9,11])
35

```


本文是為了教Functional Programming style舉出operator和reduce搭配使用

讓讀者懂得如何用這些功能

再舉一例,如果有兩向量$$V_{1},V_{2}$$ 



```python

>>> V1 = [2,3,4]
>>> V2 = [5,-2,3]

```


求兩向量內積 $$v_{11}v_{21}+v_{12}v_{22}+v_{13}v_{23}$$

可以用 `operator.add` 和 `operator.mul` ,如下：



```python

>>> reduce(operator.add,map(operator.mul, V1,V2))
16

```


#### 1.2 operator.itemgetter


在運算符號中, `operator.itemgetter` 相對於運算符號的 `[]`, 

可以取出list 或dict 中的element

但有些情形 `operator.itemgetter` 會比較方便


給定一個list


```python

>>> my_list = [1,2,1,3,4,6]

```


假如要取出某個list中index為1者



```python

>>> operator.itemgetter(1)(my_list)
2

```


這樣還是直接用 `[]` 比較快



```python

>>> my_list[1]
2

```


但是如果要取出index為1,3,5的三個element



```python

>>> my_list[1],my_list[3],my_list[5]
(2, 3, 6)

```


這樣有點麻煩了,來用 `operator.itemgetter` 看看



```

>>> operator.itemgetter(1,3,5)(my_list)
(2, 3, 6)

```


用, `operator.itemgetter` 就比較快了


至於還有哪些情況會用到 `operator.itemgetter` ?

例如在計算語言學的研究中,用freqdist計算每個單字在文章中出現的頻率,

如下, *is* 出現了193次,之類的



```python

>>> freqdist=[('a', 185), ('is', 93), ('the', 219), ('he',51)]

```


如果要取出每個單字的出現頻率,可以用 `itemgetter` 搭配 `map` 使用

如果把單字依出現頻率高低排序,可以用 `itemgetter` 搭配 `sorted` 使用



```python

>>> map(operator.itemgetter(1),freqdist)
[185, 93, 219, 51]

>>> sorted(freqdist, key=operator.itemgetter(1),reverse=True)
[('the', 219), ('a', 185), ('is', 93), ('he', 51)]

```


#### 1.3 operator.methodcaller


如果遇到一種情況,要根據某個variable的值,來判斷要call哪個function

假設有個class有兩個function,如下:



```python

>>> class CallerDemo():
...     def print_a(self):
...         print 'a'
...     def print_b(self):
...         print 'b'
... 

```


然後根據variable `x` 來判斷要call `print_a()` or `print_b()` 



```python

>>> def my_print(x):
...     if x == 'a':
...         CallerDemo().print_a()
...     elif x == 'b':
...         CallerDemo().print_b()
... 

```


執行結果如下



```python

>>> my_print('a')
a

>>> my_print('b')
b

```


這樣寫真的頗麻煩的,

這個時候可以用 `operator.methodcaller`, 根據字串名稱,選擇要call哪個function

例如:



```

>>> {'1':11,'2':22}.keys()
['1', '2']

>>> operator.methodcaller('keys')({'1':11,'2':22})
['1', '2']

```


用了 `operator.methodcaller` ,改寫之前的 `my_print(x)` ,一行就夠了



```python

>>> def my_caller_print(x):
...     operator.methodcaller("print_%s"%(x))(CallerDemo())

```


執行結果如下



```python

>>> my_caller_print('a')
a

>>> my_caller_print('b')
b

```




## 2 functools.partial


functools 是一些針對higher-order function的一個模組

這些higher-order function可以把其他function當成argument


先載入一下模組



```python

>>> import functools

```

我們先來介紹一下 `functools.partial` 的功能


假設現在要保留某個list之中等於1的element,

可以用到先前提到的 `operator` 模組,的 `operator.eq(a,b)` 來比較大小



```python

>>> x=1
>>> operator.eq(1,x)
True

```


如果要搭配 `filter` 使用,來去掉list中不等於1的element,該怎麼辦呢？

因為 `operator.eq(a,b)` 需要兩個argument, 

但 `filter` 只能接受一個function與另一個argument

如果要輸入 `eq(a,b)` 所需要的兩個argument,會出現error,如下：



```python

>>> filter(operator.eq,1,[1,2,3,1,1,2,2,1,3])
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: filter expected 2 arguments, got 3

```


這個時候,就需要用到 `functools.partial` 了

它可以先把 `operator.eq(a,b)` 先和其中一個argument做結合,包成一個function

使用方法如下：



```python

>>> eq1=functools.partial(operator.eq,1)

```


這樣產生出來的function, `eq1` 就只需要一個argument了



```python

>>> eq1(1)
True

>>> eq1(2)
False

>>> 

```


也就是說,可以把它放到 `filter` 裡面使用了



```python

>>> filter(eq1,[1,2,3,1,1,2,2,1,3])
[1, 1, 1, 1]

```


再舉一個例子,

如果要十六進位的數轉成十進位,可以用 `int()` 這個function



```python

>>> int("ABCDEF",base = 16)
11259375

>>> int("AAAAAA",base = 16)
11184810

```


這樣每次都要輸入兩個argument,而且第二個argument要一直重複輸入,比較麻煩

可以用 `functool.partial` 先把int和第二個argument包成一個,如下



```python

>>> base16 = functools.partial(int, base=16)
>>> base16("ABCDEF")
11259375

>>> base16("AAAAAA")
11184810

```


這樣子看起來就簡潔多了


## Further Reading:


其實 `functool` 還有一個很好用的function: `functool.wraps`

但這個跟 *Design Pattern* 的 *decorator* 有關

等之後有機會提到 *Design Pattern* 的時候再來講解


關於本文所講到的,想知道更多,請看：

operator: http://docs.python.org/2/library/operator.html

functools: http://docs.python.org/2/library/functools.html