---
layout: post
title: 'NLTK Logic 3 : Discourse Representation Theory'
date: 2014-03-21 03:58
comments: true
categories: ['Natural Language Processing', 'Semantics','NLTK']
---

## 1. Introduction

*Discourse* 的意思是對話

在對話中,常常會用到 **代名詞** ,像是 *he*, *she* 或 *it*.

我們把這種代名詞叫做 *anaphoric pronouns*

因為要從前面的句子去判斷,這些代名詞代表什麼

比如有個句子 *A woman walks. She smokes.*

在下一句的 *She* 是指前一句提到的 *A woman*


那要怎麼讓電腦去判斷, **代名詞** 到底代表前面提到的什麼？

這就要用到 *Discourse Representation Theory (DRT)* 來處理了

例如 *A woman walks* 這句話,用 *DRT* 可以表示成這樣：


$$

    \begin{array}{|c|}

    \hline x  \\ \hline

    	woman(x) \\

      walk(x) \\

    \hline

		\end{array}

$$

<!--more-->

這樣一個框框的結構叫作 *Discourse Representation Structure* ,簡稱 *DRS*

其中位於方框上面的 $$x$$ 叫做 *Discourse Referent* 

它代表這個句子中,可以被其他代名詞參考的東西

方框下方的式子叫做 *DRS conditions* 

它代表這些文句所產生的情境


第二個句子 *She smokes.*

用 *DRT* 表示成這樣：

$$

    \begin{array}{|c|}

    \hline y  \\ \hline

      PRO(y) \\

      smoke(y) \\

    \hline

		\end{array}

$$


其中, $$PRO(y)$$ 表示 $$y$$ 是一個代名詞,但不知道它代表誰

這個時候就要用 *DRT* ,把對話中前後語句的情境結合起來

就可以進行 *Resolve Anaphora* 找出代名詞代表什麼

例如：

*A woman walks. She smokes.*

用 *DRT* 表示成：

$$

    \begin{array}{|c|}

    \hline x  \\ \hline

    	woman(x) \\

      walk(x) \\

    \hline

		\end{array}

    +

    \begin{array}{|c|}

    \hline y  \\ \hline

      PRO(y) \\

      smoke(y) \\

    \hline

		\end{array}

    \xrightarrow{Simplify}

    \begin{array}{|c|}

    \hline x,y  \\ \hline

    	woman(x) \\

      walk(x) \\

      PRO(y) \\

      smoke(y) \\

    \hline

		\end{array}

    \xrightarrow{Resolve\mspace{5mu} Anaphora}

    \begin{array}{|c|}

    \hline x,y  \\ \hline

    	woman(x) \\

      walk(x) \\

      x = y \\

      smoke(y) \\

    \hline

		\end{array}

$$

藉由 *DRT* ,我們可以知道 *She* 是指 *A woman*

然後,還可以把 *DRS* 轉成一階邏輯的式子：

像這樣：

$$

    \begin{array}{|c|}

    \hline x,y  \\ \hline

    	woman(x) \\

      walk(x) \\

      x = y \\

      smoke(y) \\

    \hline

		\end{array}

    \Rightarrow

    \exists x \exists y.( woman(x) \wedge walk(x) \wedge (x = y) \wedge smoke(y))

$$




## 2. DRT in nltk


現在我們來用nltk實作看看

先載入模組



```python

>>> import nltk

```


接著我們輸入 *A woman walks* 的 *DRT* 



```python

>>> dp = nltk.DrtParser()
>>> drs1 = dp.parse('([x], [woman(x), walk(x)])') 
>>> print drs1
([x],[woman(x), walk(x)])

```


我們也可以把 *DRT* 的結構畫出來



```python

>>> drs1.draw()

```


像這樣


<img src="http://lh6.googleusercontent.com/-89PLq9VYopQ/UyvQx-TxTfI/AAAAAAAAAqY/cU232E7UWKQ/s165-no/DRT.png">


接著我們把第二句話 *She smokes.* 也一起輸入



```python

>>> drs2 = dp.parse('([y], [PRO(y),smokes(y)])') 
>>> drs2.draw()

```


<img src="http://lh3.googleusercontent.com/-ugJ-AhsCo28/Uyva__Uz4QI/AAAAAAAAAqw/wgYKB9yYOcI/w163-h165-no/DRT2.png">


其中 `PRO(y)` 是未知的代名詞,

因為只看這句話不知道這個代名詞代表什麼, 

需要和第一句話結合起來才知道


我們可以用 `+` 把這 *兩個DRS* 合起來



```python

>>> drs3=drs1+drs2
>>> drs3.draw()

```


<img src="http://lh6.googleusercontent.com/-ZVVEazh0bxU/Uyva_xl9HZI/AAAAAAAAArE/FOh2NjrkihE/w335-h158-no/DRT3.png">


再用 `simplify()` 把這兩個 *DRS* 簡化成一個



```python

>>> drs4=drs3.simplify()
>>> drs4.draw()

```


<img src="http://lh6.googleusercontent.com/-XS97Rkd11iM/Uyva_tAb7oI/AAAAAAAAArU/ga7l4KP5QV4/w165-h240-no/DRT4.png">


然後再用 `resolve_anaphora()` 找出代名詞 `PRO(y)` 代表什麼



```python

>>> drs5=drs4.resolve_anaphora()
>>> drs5.draw()

```


<img src="http://lh6.googleusercontent.com/-xqOOLlm0h7w/UyvbAKBA7RI/AAAAAAAAAq8/KOkSJP2nJG4/w168-h240-no/DRT5.png">


這時,原本的 `PRO(y)`  會變成 `(y = x)` 表示已經找出了 `y` 代表什麼


然後, 還可以把 *DRS* 轉成 *First-order Logic* 的式子

像這樣：



```python

>>> f1=drs5.fol()
>>> print f1
exists x y.(woman(x) & walks(x) & (y = x) & smokes(y))

```


這樣就大功告成了


## 3.Further Reading


其實 *DRT* 還可以解決許多關於對話中的語意問題

有興趣的話可以看這個網站：Discourse Representation Theory 

http://www.coli.uni-saarland.de/projects/milca/courses/comsem/html/node205.html

以及這本書：

*Hans Kamp, Josef van Genabith, Uwe Reyle. Discourse Representation Theory*
