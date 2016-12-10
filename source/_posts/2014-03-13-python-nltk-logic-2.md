---
layout: post
title: 'Python nltk -- Logic 2 : Lambda Calculus'
date: 2014-03-13 14:09
comments: true
categories: [Python, nltk, semantics, logic]
---

本篇介紹如何用python nltk 的應用,邏輯語意與lambda calculus


## 1.introduction


邏輯語意學在語意推導方面,通常會用到 $$\lambda -calculus$$

使用$$\lambda -calculus$$就可以把一個句子的語意,從個別單字中推導出來

至於$$\lambda -calculus$$ 是什麼呢？

簡而言之,lambda calculus是一種數學運算,由以下三種元素組成

$$

\begin{align}


& v  & (a)\\

& \lambda x.f(x)  & (b)\\

& \lambda x.f(x)(v) & (c)\\


\end{align}

$$

$$(a)$$ 是 $$variable$$

$$(b)$$ 是 $$astraction$$ ,就是把function中的variable拿到前面,加個 $$\lambda$$

$$(c)$$ 是將另一個 $$variable$$ 放到 $$astraction$$ 後面

然後可以進行一種運算,叫做 $$\beta-reduction$$ ,如下

$$

\lambda x.f(x)(v)

\Rightarrow f(v)

$$

還有另一種運算叫做 $$\alpha-conversion$$ ,其實就是更改變數名稱而已

$$

\lambda x.f(x)

\Rightarrow \lambda y.f(y)

$$

<!--more-->

$$\lambda -calculus$$ 也是計算理論的一種,和Turing Machine等價

實際上以 $$\lambda -calculus$$ 實現的程式語言有Functional Programming,例如Lisp或Haskell


## 2. lambda expression  


接著來實作untyped lambda calculus,

首先,載入模組 `nltk.sem.logic` 和 `LogicParser`



```python

>>> import nltk.sem.logic as logic
>>> lgp = logic.LogicParser()

```


再看看 $$\lambda$$ 要用哪個符號



```python

>>> logic.binding_ops()
existential    	exists
universal      	all
lambda         	\

```


要表示lambda $$\lambda$$ 符號,要用 `\` 字串

然後,用`LogicParser`來輸入這個式子：$$e1=\lambda x.P(x)$$



```python

>>> e1 = lgp.parse(r'\x.P(x)')
>>> print e1
\x.P(x)

```


## 3. alpha conversion


再來試試看$$\alpha-conversion$$ : $$\exists x.P(x) \Rightarrow \exists z.P(z)$$

從以下結果得知,$$\alpha-conversion$$ 轉換前和轉換後,在邏輯上是相等

因為只是更改變數名稱而已



```python

>>> e2 = e1.alpha_convert(logic.Variable('z'))
>>> print e2
\z.P(z)

>>> e1 == e2
True

```


## 4. beta reduction


接著,來看看 $$\beta-reduction$$ 

舉一個自然語言的例子,例如 *Gary walks.* 這個句子,用邏輯語意可以表示成

$$

walks(Gary)

$$

根據Principle of Compositionality,

句子的語意是由單字的語意所組成的(但實際上還是有例外,例如片語)

推導出這個句子的語意,可由單字的語意推導出來,

*walks* 的語意是 $$\lambda x.walk(x)$$

*Gary* 的語意是 $$Gary$$

則 *Gary walks* 的語意是

$$

\lambda x.walks(x) (Gary)\Rightarrow walks(Gary)

$$

接著來實作看看



```python

>>> e3 = lgp.parse(r'\x.walks(x)(Gary)')
>>> e4 = e3.simplify()
>>> print e4
walks(Gary)

```


再舉一個例子,例如 *Gary sees Mary.* 這個句子, *sees* 是及物動詞

及物動詞,是表示一個主詞和受詞的關係,

可用 *First Order Logic* 的 *Relation* 來表示

*sees* 的語意為 $$\lambda x \lambda y .see(x,y)$$

而 *Gary sees Mary.* 的語意是

$$

\begin{align}

& \lambda x \lambda y.see(x,y) (Gary) (Mary) \\

& \Rightarrow \lambda y(Gary,y) (Mary) \\

& \Rightarrow see(Gary,Mary)

\end{align}

$$


接著,輸入nltk看看



```python

>>> e5 = lgp.parse(r'\x \y.see(x,y)(Gary)(Mary)')
>>> print e5
((\x y.see(x,y))(Gary))(Mary)

>>> print e5.simplify()
see(Gary,Mary)

```


還有其他的輸入方式,結果都一樣



```python

>>> e6 = lgp.parse(r'\x \y.see(x,y)(Gary,Mary)')
>>> e7 = lgp.parse(r'\x y.see(x,y)(Gary,Mary)')
>>> e8 = lgp.parse(r'\x y.see(x,y)(Gary)(Mary)')
>>> e9 = lgp.parse(r'(\x y.see(x,y)(Gary))(Mary)')

```


結果如下：



```python

>>> print e6
((\x y.see(x,y))(Gary))(Mary)

>>> print e7
((\x y.see(x,y))(Gary))(Mary)

>>> print e8
((\x y.see(x,y))(Gary))(Mary)

>>> print e9
((\x y.see(x,y))(Gary))(Mary)

>>> e6 == e7 == e8 == e5
True

```





## 5.further reading


想要瞭解 $$\lambda -calculus$$, 請參考http://en.wikipedia.org/wiki/Lambda_calculus

想要看看 nltk 裡面的 $$\lambda -calculus$$ , 請至http://www.nltk.org/howto/logic.html

還有關於邏輯語意的研究,可以參考這本書 *Blackburn & Bos'  Representation and Inference for Natural Language*
