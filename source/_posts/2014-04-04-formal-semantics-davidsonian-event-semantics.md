---
layout: post
title: 'Davidsonian Event Semantics'
date: 2014-04-04 17:13
comments: true
categories: ['Natural Language Processing', 'Semantics']
---

## 1.Introduction


所謂的形式語義學( *Formal Semantics* ), 是在研究, 如何把自然語言用邏輯形式來表達


例如以下句子

$$

\begin{align}

&\text{John buttered the toast.} &(1.a)

\end{align}

$$

傳統上, 用一皆邏輯 *First Order Logic* 可以把這個句子表示成這樣

$$

\begin{align}

&butter(John,toast) &(1.b)

\end{align}

$$

<!--more-->


其中, *butter* 這個邏輯式的predicate, 而 *John* 和 *toast* 分別是argument, 這個邏輯式表示了 *John* 和 *toast* 的關係, 

當這句話為真的時候, 

$$

butter(John,toast) \equiv True

$$


如果現在句子是這樣, 有個介繫詞片語修飾, 例如：

$$

\begin{align}

&\text{John butterd the toast with the knife} &(2.a) 

\end{align}

$$

則可以把 $$(1.b)$$ 的一階邏輯式,再加入一個論元 (argument) 擴充成這樣

$$

\begin{align}

&butter(John,toast,knife) &(2.b) 

\end{align}

$$


但這方法有個缺點, 就是無法推論 $$(2.b)$$ 這個邏輯式是否蘊含 $$(1.b)$$

$$

butter(John,toast,knife) \nrightarrow butter(John,toast)

$$

而事實上, $$(2.a)$$ 這句話蘊含 $$(1.a)$$ , 是成立的

$$

\text{John butterd the toast with the knife} \rightarrow \text{John buttered the toast.}

$$


照理說, 邏輯式要可以表達這些自然語言中的蘊含關係

因此就要想出一個邏輯表達方法, 也可以讓這些蘊含推論成立





## 2.Davidsonian event semantics



*Davidson* 在 1969 年提出了 *event* 的概念 



> what adverbial clauses modify is not verbs but the events that certain verbs introduce.
> Davidson (1969/1980: 167)


意思就是, 修飾動詞的修飾語, 其實是在修飾這個動詞引出的事件

例如, 針對句子 $$(1.a)$$  , 可以把動詞 *butter* 引出的事件定為 $$e$$ , 並加到 $$(1.b)$$ 中 *butter*的argument,如下：


$$

\begin{align}

&\exists e  (butter(John,toast,e)) &(1.c)

\end{align}

$$


其中, 式子前面的 $$\exists e$$ 表示, 當這個式子成立的時候, 存在 $$e$$ 這樣一個事件


然後, 句子 $$(2.a)$$ 的邏輯式也可以用 *event* 的概念擴充, 可以把修飾語 *with a knife* 替換成 $$instr(e,knife)$$,  表示執行這個 *event* 的 *instrument* 為 *knife* , 如下：


$$

\begin{align}

&\exists e  (butter(John,toast,e)\wedge instr(e,knife)) &(2.c)

\end{align}

$$



來檢查一下邏輯式 $$(2.c)$$ 是否蘊含 $$(1.c)$$

$$

\exists e  (butter(John,toast,e)\wedge instr(e,knife)) \rightarrow \exists e  (butter(John,toast,e))

$$


以上蘊含成立


除此之外, 一個動詞可能同時被很多個介繫詞片與修飾, 例如以下句子


$$

\begin{align}

\text{John buttered the toast in the bathroom with the knife at midnight} &(3.a)

\end{align}

$$


$$(3.a)$$ 不但蘊含 $$(2.a)$$ 也蘊含 $$(1.a)$$


$$

\begin{align}

&\text{John buttered the toast in the bathroom with the knife at midnight} &(3.a) \\

&\rightarrow  

\text{John butterd the toast with the knife} &(2.a)  \\

&\rightarrow 

\text{John buttered the toast.} &(1.a) \\

\end{align}

$$


這種句子如果用沒有引入 *event* 的概念, 用 $$(2.b)$$ 的傳統邏輯式子, 問題就更大了


$$

\begin{align}

&butter(John,toast,knife,bathroom,midnight) &(3.b) 

\end{align}

$$

這樣的話, 我們就要定義 *butter* 可以接收很多個argument, 且要定好哪個位置是填給哪一種修飾語用的, 當然, $$(3.b)$$ 蘊含 $$(2.b)$$ 的推論也不成立


用 *event* 的概念就簡單很多了, 不用修改到 *butter* 的argument, 只要把修飾語轉換成修飾 $$e$$ 的predicate, 並和 *butter* 做conjunction, 就可以了, 如下


$$

\begin{align}

&\exists e  (butter(John,toast,e)\wedge instr(e,knife) \wedge in(e,bathroom) \wedge at(e,midnight) ) &(3.c)

\end{align} 

$$


而且 $$(3.c)$$ 蘊含 $$(2.c)$$ 也蘊含 $$(1.c)$$ 


$$

\begin{align}

&\exists e  (butter(John,toast,e)\wedge instr(e,knife) \wedge in(e,bathroom) \wedge at(e,midnight) ) &(3.c) \\

&\rightarrow \exists e  (butter(John,toast,e)\wedge instr(e,knife)) &(2.c) \\

&\rightarrow \exists e  (butter(John,toast,e)) &(1.c) \\

\end{align}

$$


## 3.Implementation


來載入模組, 實作看看



```python

>>> import nltk.sem.logic as logic
>>> from nltk import Prover9
>>> lgp = logic.LogicParser()

```


用前面提到的句子和邏輯式子來做練習


$$

\begin{align}

&\text{John buttered the toast.} &(1.a) \\

&\text{John butterd the toast with the knife} &(2.a)  \\

&\text{John buttered the toast in the bathroom with the knife at midnight} &(3.a) \\

\\

& \exists e  (butter(John,toast,e)) &(1.c) \\

& \exists e  (butter(John,toast,e)\wedge instr(e,knife)) &(2.c) \\

&\exists e  (butter(John,toast,e)\wedge instr(e,knife) \wedge in(e,bathroom) \wedge at(e,midnight) ) &(3.c) \\

\end{align}

$$


把 $$(1.a)$$ , $$(2.a)$$ , $$(3.a)$$ , 這三句話的邏輯式  $$(1.c)$$ , $$(2.c)$$ , $$(3.c)$$ 分別輸入到LogicParser, 如下：



```python

>>> a1 = lgp.parse('exists e ( butter(John,toast,e))')
>>> a2 = lgp.parse('exists e ( butter(John,toast,e) & instr(e,knife) )')
>>> a3 = lgp.parse('exists e (butter(John,toast,e) & instr(e,knife) & in (e,bathroom) & at(e,midnight) )')

```


接著來看看句子 $$(1.a)$$ 和 $$(2.a)$$ 的蘊含關係

用 `Prover9()` 來證明, 我們把未知值的句子放在第一個argument ,  已知為真的句子放在第二個argument中

例如, 想要證明當  $$(2.a)$$ 為真時, $$(1.a)$$ 是否為真, 輸入方法如下：



```python

>>> Prover9().prove(a1, [a2])
True

```


結果顯示, 當 $$(2.a)$$ 為真, $$(1.a)$$ 必為真, 表示 $$(2.a)$$  蘊含 $$(1.a)$$ 


把 $$(2.a)$$ 和 $$(1.a)$$ 對調



```python

>>> Prover9().prove(a2, [a1])
False

```


結果顯示, $$(1.a)$$ 蘊含 $$(2.a)$$ 不成立



然後來看看 $$(3.a)$$ 和 $$(2.a)$$ 以及 $$(1.a)$$ 的蘊含關係



```python

>>> Prover9().prove(a2, [a3])
True

>>> Prover9().prove(a1, [a3])
True

```


結果顯示, $$(3.a)$$ 蘊含 $$(2.a)$$ 也蘊含 $$(1.a)$$


## 4. Reference


本文參考至以下這本語意學書籍

[Semantics: An International Handbook of Natural Language Meaning](http://www.amazon.com/Semantics-International-Handbooks-Linguistics-Communication/dp/3110184702)
