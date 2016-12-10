---
layout: post
title: 'Python nltk -- Logic 1 : First Order Logic and Semantics'
date: 2014-03-13 13:53
comments: true
categories: 
---

本篇介紹如何用python nltk 的應用,邏輯語意與推理


## 1. Introduction

邏輯語意學(logic semantics/formal semantics)

是早期的自然語言處理方法之一

因為自然語言中存在著某種邏輯成份

故可以用數理邏輯的公式來描述自然語言

進行各種邏輯推理


例如 *Socrates is a man.* 這句話,用一階邏輯式表示:

$$

man(Socrates)

$$

如果 *Socrates is a man.* 這句話為真,則:

$$

man(Socrates)= true 

$$

<!--more-->

另一個例子,如 *All man are mortal* 這句話,牽涉到 *All* 這個概念,用一階邏輯式表示:

$$

\forall x .(man(x) \rightarrow mortal(x)) 

$$

接著,可以來用演繹法(Deductive Reasoning)進行推論

根據 *Socrates is a man.* 和 *All man are mortal* 這兩句話

可以推論出 *Socrates is mortal* 

$$

man(Socrates) \wedge \forall x .(man(x) \rightarrow mortal(x))  \Rightarrow mortal(Socrates)


$$


## 2. First-order Logic Expression


#### 2.1 Logic Symbol


接著來實作logic inference,

首先,載入模組 `nltk.sem.logic` 和 `LogicParser`



```python

>>> import nltk.sem.logic as logic
>>> lgp = logic.LogicParser()

```


接著,在 `nltk.sem.logic` 中要用哪些符號,來表示邏輯運算符號

根據以下結果,就可以知道了

例如,要表示exists $$\exists$$ 符號,要用 `exists` 字串



```python

>>> logic.boolean_ops()
negation       	-
conjunction    	&
disjunction    	|
implication    	->
equivalence    	<->

>>> logic.equality_preds()
equality       	=
inequality     	!=

>>> logic.binding_ops()
existential    	exists
universal      	all
lambda         	\

```


然後,我們來看看如何輸入一階邏輯式的組成

一階邏輯式可以分為 *Function* 和 *Argument* 兩部份

例如：

$$

man(Gary)\\

woman(x)

$$

*man* 和 *woman* 是 *Function* ,而 *Gary* , *x* 是 *Argument*  

而 *Argument* 的部份,可以是 *Constant* 也可以是 *Variable*

例如 *Gary* 是 *Constant* ,是已知的值,不能被代換成其他的值

而 *x* 是 *Variable* ,是未知的值,可以被帶換成其他的值


把以上邏輯式子輸入進去



```python

>>> e1 = lgp.parse('man(Gary)')
>>> e2 = lgp.parse('woman(x)')
>>> print e1
man(Gary)

>>> print e2
woman(x)

```


接著看看它們的 *Function* 和  *Argument* 



```python

>>> e1.function
<ConstantExpression man>

>>> e2.function
<ConstantExpression woman>

>>> e1.argument
<ConstantExpression Gary>

>>> e2.argument
<IndividualVariableExpression x>

```


一階邏輯式子中也可以有 *quantifier* 的組成,例如 $$\forall$$是 *quantifier*


$$

\forall y .(man(y) \leftrightarrow \neg woman(y))

$$


然後,輸入以上式子



```python

>>> e3 = lgp.parse('all y.(man(y) -> -woman(y))')
>>> print e3
all y.(man(y) -> -woman(y))

```


學會了以上這些,就可以輸入各種邏輯符號到nltk裡了


## 3. Theorem Proving


現在來看看怎麼用nltk做以下推理

$$

man(Socrates) \wedge \forall x .(man(x) \rightarrow mortal(x))  \Rightarrow mortal(Socrates)

$$

首先,把前提和結論,由parser輸入



```python

>>> p1 = lgp.parse('man(Socrates)')
>>> p2 = lgp.parse('all x.(man(x) -> mortal(x))')
>>> c  = lgp.parse('mortal(Socrates)')

```


接著要用Theorem Prover來證明這個推導成立

Theorem Prover是一種SAT solver

只要給定一些邏輯式

它就可以判斷這些邏輯式之間可不可以被滿足,有沒有矛盾


在nltk有提供很多種Theorem Prover的界面

例如Prover9, TableauProver, and ResolutionProver

但是只有提供界面而已,prover的核心程式還是要自己去下載


在這篇,我選用的是Prover9(其實用哪一種都可以）

首先,我們要先到此下載prover9, http://www.cs.unm.edu/~mccune/prover9/

下載好之後,請看compile的方法http://www.cs.unm.edu/~mccune/prover9/manual/2009-11A/

compile好以後, 丟到 `/usr/local/bin` 目錄底下

（由於我都只用Linux或Mac系統,如果你使用Windows的話,我無法告訴你怎麼安裝,真抱歉）


接著我們用nltk的界面來呼叫prover9



```python

>>> from nltk import Prover9
>>> Prover9().prove(c, [p1,p2])
True

```


結果為True, 表示$$[p1,p2]\Rightarrow c$$的推導成立

但如果你執行prover9後顯示出這樣：



```python

>>> Prover9().prove(c, [p1,p2])
which: no prover9 in (/usr/lib/jvm/java-1.7.0/bin:/usr/local/eclipse:/opt/ant/bin:/usr/lib64/qt-3.3/bin:/usr/local/maven/bin:/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/sbin:/usr/local/android-sdk-linux:......

```


這是因為prover沒有放到指定的資料夾


來看一個推理結果為False的例子：


$$

man(Gary) \wedge \forall x .(man(x) \rightarrow \neg woman(x))  \Rightarrow 

\neg woman(Gary)

$$


把前提和結論輸入



```python

>>> e1 = lgp.parse('man(Gary)')
>>> e3 = lgp.parse('all y.(man(y) -> -woman(y))')
>>> c1= lgp.parse('woman(Gary)')
>>> Prover9().prove(c1, [e1,e3])
False

```


當然,如果把結論改為 $$\neg woman(Gary)$$

則結果為True



```python

>>> c2=c1.negate()
>>> print c2
-woman(Gary)

>>> Prover9().prove(c2, [e1,e3])
True

```



## 4.Further Reading

關於邏輯語意,可到這網站 http://www.coli.uni-saarland.de/projects/milca/courses/comsem/html/

或參考這本書 *Blackburn & Bos' Representation and Inference for Natural Language*

還有更多關於nltk的邏輯推理 http://www.nltk.org/howto/inference.html
