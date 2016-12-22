---
layout: post
title: 'NLTK Logic 4 : Model and Satisfiability'
date: 2014-03-29 15:39
comments: true
categories: ['Natural Language Processing', 'Semantics','NLTK']
---

## 1.Model and Satisfiability


可滿足性(Satisfiability)是在探討, 邏輯式子所建立出的模型(Model),

可不可以找到一組解, 使得這個 *Model* 算出來的值可以是 $$True$$

例如：

$$

p1=(x \vee y), \mspace{5mu}

p2=(\neg x \vee y),  \mspace{5mu}

p3=(x \vee \neg y) \\

M1=p1 \wedge p2 \wedge p3

$$

則當 $$x\equiv True, \mspace{5mu} y \equiv  True$$ 時, $$M1 \equiv True $$

則 *Model* $$M1$$ 是 *Satisfiable*

另一例子：

$$

p1=(x \vee y), \mspace{5mu}

p2=(\neg x \vee y),  \mspace{5mu}

p3=(x \vee \neg y) \mspace{5mu}

p4=(\neg x \vee \neg y)\\

M2=p1 \wedge p2 \wedge p3 \wedge p4

$$

這種情形,不論 $$x$$ 或 $$y$$ 的值,  $$M2$$ 永遠都是 $$False$$

則 *Model* $$M2$$ 是 *Unsatisfiable*

<!--more-->

## 2.Implementation 1


接著我們用nltk來做簡單的 *Satisfiability* 問題

先載入模組



```python

>>> import nltk
>>> lgp = nltk.LogicParser()

```


接著,輸入邏輯式子$$p1=(x \vee y), \mspace{5mu} p2=(\neg x \vee y),  \mspace{5mu}p3=(x \vee \neg y) \mspace{5mu}$$



```python

>>> p1 = lgp.parse('x | y')
>>> p2 = lgp.parse('-x | y')
>>> p3 = lgp.parse('x | -y')

```


然後把assignment加進去, 看看 $$M1=p1 \wedge p2 \wedge p3$$ 是否 *satisfiable*


```python

>>> val = nltk.Valuation([('x', True), ('y', True)])
>>> dom = val.domain
>>> g = nltk.Assignment(dom)
>>> m = nltk.Model(dom, val)
>>> m.satisfy(p1 & p2 & p3, g ) 
True

```


再把 $$p4=(\neg x \vee \neg y)$$ 加進去, 看看$$M2=p1 \wedge p2 \wedge p3 \wedge p4$$ 是否 *satisfiable*




```python

>>> p4 = lgp.parse('-x | -y')
>>> m.satisfy(p1 & p2 & p3 & p4, g ) 
False

```


## 3. First Order Logic


接著我們來看看如何將 *Model* 和 *Satisfiability* 的概念, 推廣到一階邏輯(First Order Logic)中

例如,在某個世界裡,

*Gary* 是個男生, *Judy* 是個女生, 而 *Henry* 是一隻狗,

*Gary* 和 *Henry* 在跑步, 

*Judy* 看見 *Henry* 而 *Gary* 看見 *Judy*

我們可以用一階邏輯(First Order Logic)把這個世界表示成一個 *Model*：

$$

\begin{align}

&M=\\

& Gary(g)  &\wedge \mspace{15mu} & Judy(j)  &\wedge \mspace{15mu} &Henry(h) &\wedge\\

& Boy(g)   &\wedge \mspace{15mu} & Girl(j)  &\wedge \mspace{15mu} &Dog(h)   &\wedge\\

& Run(g)   &\wedge \mspace{15mu} & Run(h)   &\wedge \mspace{15mu} & & \\

& See(j,h) &\wedge \mspace{15mu} & See(g,j) &\wedge \mspace{15mu} &See(g,h)

\end{align}

$$


根據這個Model, 用*Satisfiability*的概念, 可以求解以下問題

1 . 找找看這個世界中女生有哪些

這個問題可以看成是, 從找 *M* 中找出滿足 $$girl(x)$$ 的 $$x$$ 值有哪些, 

可求解 $$girl(x) \wedge M $$ 的 *Satisfiability* , 得出：

女生有 *Judy* , 因為當 $$x=j$$ 時, $$girl(x) \wedge M $$ 為 *Satisfiable*


2 . 回答這個世界中是否有男生在跑步, 或者, 是否有女生在跑步

這可以看成是, 判斷某個 formula 和這個 *M* 是否有交集, 得出：

有男生在跑步, 因為 $$\exists x(Run(x) \wedge boy(x) ) $$ 在 $$ M $$ 中為 *Satisfiable*

沒有女生在跑步, 因為 $$\exists x(Run(x) \wedge girl(x) ) $$ 在 $$ M $$ 中為 *Unsatisfiable*


3 . 回答是否 *Judy* 看見 *Henry* , 或者,是否 *Judy* 看見 *Gary*

可以判斷某個assignment 是否可讓 *M* 為 *Satisfiable* , 得出：

*Judy* 看見 *Henry* , 因為當 $$x=j,y=h$$ 時, $$See(x,y) $$ 在 $$M $$ 中為 *Satisfiable*

*Judy* 沒有看見 *Gary* , 因為當$$x=j,y=g$$ 時, $$See(x,y) $$ 在 $$M $$ 中為 *Unsatisfiable*


註：

1.在 python nltk 用的是 *Closed World Assumption* 

也就是說, 在 *Model* 裡面沒有定義的, 一律為 *False*

例如沒有定義 *Run(j)* , 所以 *Run(j)* 為 *False* 

因此 $$\exists x(Run(x) \wedge girl(x) ) $$ 在 $$ M $$ 中為 *Unsatisfiable* 

同理, *See(j,g)* 在 $$M $$ 中為 *Unsatisfiable*


2.相對於 *Closed World Assumption* 

有另一種叫作 *Open World Assumption* 

是將沒有定義的formula 它視為 *Unknown*

所以結果除了 *True* 和 *False* 之外, 還有 *Unknown*


## 4.Implementation 2


用 python nltk 來實作看看：

首先, 建立 model



```python

>>> v = """ 
... Gary => g
... Judy => j
... Henry => h
... boy => {g}
... girl => {j}
... dog => {h}
... run => {g, h}
... see => {(j, h), (g, j), (g, h)}
... """

>>> val = nltk.parse_valuation(v)
>>> m = nltk.Model(val.domain, val)

```


1.求 $$girl(x)$$ 的 $$x$$ 值:



```python

>>> m.satisfiers(lgp.parse('girl(x)'), 'x'  , nltk.Assignment(val.domain, []))
set(['j'])

```


得出結果 $$x=j$$


2.求 $$\exists x (Run(x) \wedge boy(x)) $$ 或 $$\exists x (Run(x) \wedge girl(x))$$ 這兩個formula在 $$M$$ 中是否為 *Satisfiable*



```python

>>> m.satisfy(lgp.parse('exists x.(run(x) & boy(x))'),nltk.Assignment(val.domain, []))
True

>>> m.satisfy(lgp.parse('exists x.(run(x) & girl(x))'),nltk.Assignment(val.domain, []))
False

```


得出結果：

$$\exists x(Run(x) \wedge boy(x) )$$ 在 $$M $$ 中為 *Satisfiable*

$$\exists x(Run(x) \wedge girl(x) ) $$ 在 $$M $$ 中為 *Unsatisfiable*


3.求 $$x=j,y=h$$ 或 $$x=j,y=g$$ 對於 $$See(x,y)$$ 在 $$M$$ 中是否為 *Satisfiable*



```python

>>> m.satisfy(lgp.parse('see(x,y)'),nltk.Assignment(val.domain,[('x', 'j'), ('y', 'h')]))
True

>>> m.satisfy(lgp.parse('see(x,y)'),nltk.Assignment(val.domain,[('x', 'j'), ('y', 'g')]))
False

```


得出結果：

當 $$x=j,y=h$$ 時, $$See(x,y) $$ 在 $$M $$ 中為 *Satisfiable*

當$$x=j,y=g$$ 時, $$See(x,y)$$ 在 $$M $$ 中為 *Unsatisfiable*


## 5.Reference


關於python nltk的 *model* 和 *satistiability* 可參考：

http://nltk.googlecode.com/svn/trunk/doc/book/ch10.html


關於 *Closed World Assumption* 與 *Open World Assumption* 可參考：

http://en.wikipedia.org/wiki/Closed_world_assumption

http://en.wikipedia.org/wiki/Open_world_assumption
