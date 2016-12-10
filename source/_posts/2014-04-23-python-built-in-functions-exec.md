---
layout: post
title: 'Python Built-in Functions -- eval and execute'
date: 2014-04-23 01:59
comments: true
categories: [Python, eval, exec]
---

## 1.Introduction


如果要把 *string* 的內容, 當成程式碼來執行, 可以用到 *eval* 或 *exec*


例如有個 *string* , 為 `s1="3+5"` 我們想要算它執行的結果, 可用



```python

>>> s1="3+5"
>>> eval(s1)
8

```


來看一下怎麼用 *eval* 或 *exec* 


## 2. eval


`eval` 是當我們要計算某一個字串中的運算, 並且 **會回傳計算結果** ,如下



```python

>>> eval('3+1')
4

```


<!--more-->

除了用於數字, 也可用於其他 *type* , 例如 *list*



```python

>>> eval('[3,4,5]+[2]')
[3, 4, 5, 2]

```


或是我們要呼叫 *object* 的 *method* 也可以



```python

>>> eval('"234232".replace("2","@")')
'@34@3@'

```


### 2.1. eval with variable and function


*eval* 可以用於自訂變數



```python

>>> x = 3
>>> eval('x+2')
5

````


或是自訂的 *function*



```python

>>> def myfun(y):
...     return y+4
... 

>>> eval('myfun(x)')
7

```


也可用於自訂變數的 *method*



```python

>>> a=[1,2]
>>> eval('a.reverse()')
>>> print a
[2, 1]

```


### 2.2. eval with builtins


當然, *eval* 也可用於 *built-in function*



```python

>>> eval('abs(-3)')
3


```


也可以限制 *built-in function* 的使用

例如在 *eval* 的第二個 *argument* 輸入 `{"__builtins__" : None }` , 如下



```python

>>> eval('abs(-3)',{"__builtins__" : None })
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<string>", line 1, in <module>
NameError: name 'abs' is not defined

```


### 2.3. eval with restricted variable


為了避免 *eval* 不小心變更到程式裡面的某些 *variable* 的值, 我們可以用以下方法, 限制 *eval* 可以使用的 *variable* 


例如, 有三個 *variable* , 分別為 `pb1` , `pb2` 與 `pr1` , 其中, 我們只希望 *eval* 可以使用 `pb1` 和 `pb2` 兩個 *variable* , 則可以建立一個 `dict` 來限制, 如下



```python

>>> pb1 = 123
>>> pb2 = [12,13,14]
>>> pr1 = 456
>>> pblist = ['pb1','pb2']
>>> pbdict = dict([ (k, locals().get(k, None)) for k in pblist ])
>>> print pbdict
{'pb1': 123, 'pb2': [12, 13, 14]}

```


其中, `pbdict` 限制了 *eval* 可以使用哪些 *variable* , 把 `pbdict` 放在 *eval* 的第三個 *argument* , 如下



```python

>>> eval("pb1+2", None, pbdict)
125

>>> eval("pb2[2]+3", None, pbdict)
17

```


結果顯示, 有在 `pbdict` 裡面的 *variable* 可以使用, 但沒在 `pbdict` 裡面的 *variable* 就不可以使用了, 如下



```python

>>> eval("pr1+3", None, pbdict)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<string>", line 1, in <module>
NameError: name 'pr1' is not defined

```


### 2.4. restriction on eval


並不是所有的*expression* 都可以用 *eval* , 例如, *assignment* 就不能用於 *eval* , 如下



```

>>> eval('a=3')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<string>", line 1
    a=3
     ^
SyntaxError: invalid syntax

```


或是其他跟 *assignment* 有關的 *operator* 也不行



```python

>>> a=[1,2]
>>> eval('a+=[3]')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<string>", line 1
    a+=[3]
      ^
SyntaxError: invalid syntax

```


這時就需要用到 *exec* 了


## 3. exec


*exec* 是把字串當成程式碼來執行, 但是 **不會回傳結果** , 如下



```python

>>> exec('3+5')

```


如果要印出結果, 則要加個 *print*



```python

>>> exec('print 3+5')
8

```


### 3.1. exec with variable


*exec* 也可以用於 *variable*



```python

>>> x = 3
>>> exec('x+3')
>>> print x
3

```


也可用於自訂的 *function*



```python

>>> def myfun(y):
...     return y+4
... 

>>> exec('myfun(x)')
>>> print x
3

```


和 *eval* 不同的是, *exec* 可以用於 *assignment*



```python

>>> exec('z = 5')
>>> print z
5

```


以及其他和 *assignment* 相關的 *operator*



```python

>>> exec('x += 1')
>>> print x
4

```


### 3.2. exec with builtins


*exec* 可以用於 *built-in function* 



```python

>>> exec("print abs(-3)")
3

```


也可以用 `{"__builtins__" : None }` 限制 *built-in function* 的使用



```python

>>> exec("print abs(-3)",{"__builtins__" : None })
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<string>", line 1, in <module>
NameError: name 'abs' is not defined

```


### 3.3. exec with restricted variable


和 *eval* 一樣, *exec* 也可以用 *dict* 來限制可以使用的 *variable*



```python

>>> pb1 = 123
>>> pb2 = [12,13,14]
>>> pr1 = 456
>>> pblist = ['pb1','pb2']
>>> pbdict = dict([ (k, locals().get(k, None)) for k in pblist ])
>>> print pbdict
{'pb1': 123, 'pb2': [12, 13, 14]}

```


同時, *exec* 又可以使用 *assignment*, 但是要注意了, 此 *assignment* 只會修改到 *dict* 中的值, 例如我們 *assign* 一個新的值給 `pb1`



```python

>>> exec("pb1=pb1+2", None, pbdict)
>>> exec("print pb1", None, pbdict)
125

```


但是, *exec* 不會更改到原本的 *variable* 的值, 原本的 `pb1` 還是不便



```python

>>> print pb1
123

```


改變的只有在 `pdict` 中的 `pb1`



```python

>>> print pbdict['pb1']
125

```


但如果是針對 *object* 就不一樣了, 因為在 *dict* 中的是 *reference* 而不是 *value* , 所以會修改到原本 *variable* 的 *value*, 例如 *assign* 新的值給 `pb2`



```python

>>> exec("pb2[2]=pb2[2]+3", None, pbdict)
>>> print pb2
[12, 13, 17]

>>> print pbdict['pb2']
[12, 13, 17]

```



不論是原本的 `pb2` 還是 `pbdict` 中的 `pb2` 都改變了


當然, 跟 *eval* 一樣的是, 如果 *variable* 沒在 *dict* 中, 就不可以使用了



```python

>>> exec("print pr1+3", None, pbdict)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<string>", line 1, in <module>
NameError: name 'pr1' is not defined

```


### 3.4. exec with exec-defined variable


承上, *exec* 由於有 *assignment* 的功能, 所以可以自己加 *variable* 到 *dict* 裏面, 如下, 我們限制 *exec* 從一個空的 *dict*  `pbdict2={}` 開始 



```python

>>> pbdict2={}
>>> exec("x=5", None, pbdict2)
>>> exec("y=3", None, pbdict2)

```


做完以上 *assignment* 之後 , 我們再把 `pbdict` 印出來看看, 發現多了兩個新的 *variable*



```python

>>> print pbdict2
{'y': 3, 'x': 5}

```


當然, *exec* 可以用這兩個 *variable* 做運算



```python

>>> exec("print x+y", None, pbdict2)
8

```


### 3.5. exec lines of codes


*exec* 可以執行多行字串中的程式碼, 如下



```python

>>> codes="""
... x=5
... for i in range(3):
...     x+=i
...     print i,x
... """

>>> exec(codes)
0 5
1 6
2 8

```


## 4. Reference


### eval

https://docs.python.org/2/library/functions.html#eval


### exec

https://docs.python.org/2/reference/simple_stmts.html#exec