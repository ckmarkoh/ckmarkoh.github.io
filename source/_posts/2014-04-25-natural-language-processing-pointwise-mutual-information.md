---
layout: post
title: '自然語言處理 -- Pointwise Mutual Information'
date: 2014-04-25 17:58
comments: true
categories: [natural_language_processing, mutual_information]
---

## 1.Introduction


在自然語言處理中, 想要探討兩個字之間, 是否存在某種關係, 例如某些字比較容易一起出現, 這些字一起出現時, 可能帶有某種訊息


例如, 在新聞報導中, 有 *New* , *York*  , 這兩個字一起出現, 可以代表一個地名 *New York*  , 所以當出現了 *New* 這個字, 則有可能出現 *York* 


這可以用 *Pointwise Mutual Information(PMI)* 計算


*Pointwise Mutual Information* 的公式如下：


$$

pmi(x,y)=log \frac{P(x,y)}{P(x) \times P(y)}

$$

<!--more-->

其中, $$P(x,y)$$ 代表文字 *x* 和文字 *y* 一起出現的機率, 而 $$P(x)$$ 為文字 *x* 出現的機率 , $$P(y)$$ 為文字 *y* 出現的機率


如果某兩個字的出現是獨立事件, 則 *PMI* 為 *0*


$$

\begin{align}

& P(x,y) = P(x) \times P(y)\\

& pmi(x,y)=log \frac{P(x,y)}{P(x) \times P(y)} = log \frac{P(x) \times P(y)}{P(x) \times P(y)} =log1=0

\end{align}

$$


若有兩個字出現的機率不是獨立事件, 某個字出現時提昇另一個字的出現的機率, 則 *PMI* 大於 *0*


$$

\begin{align}

& P(x,y) > P(x) \times P(y)\\

& pmi(x,y)=log \frac{P(x,y)}{P(x) \times P(y)} > 0

\end{align}

$$


也就是說, 如果這兩個字的出現越不是偶然, 則 *Pointwise Mutual Information* 算出來的值越高, 越有可能帶有某種訊息



舉個例子, 以下為一個表格, 統計一篇文章中, 某些字是否一起出現

表格中的數字, 代表左方的字和上方的字, 一起出現的次數


$$

    \begin{array}{|c|c|}

    \hline            &  \text{computer}  &  \text{data}  & \text{pinch}  & \text{result}  & \text{sugar} \\

\hline \text{aprocot}    &  0        &    0    &   1    &     0   &     1 \\

\hline \text{pineapple}  &  0        &    0    &   1    &     0   &     1 \\

\hline \text{digital}    &  2        &    1    &   0    &     1   &     0 \\

\hline \text{information} &  1       &    6    &   0    &     4   &     0 \\

\hline

	\end{array}

$$


如果這篇文章總共只有 *19* 個字, 來計算 *information* 和 *data* 這兩字的 *PMI* , 得出

$$

\begin{align}

& P(x = information,y=data) 

\mspace{5mu}=\mspace{5mu} \frac{6}{19} 

\mspace{5mu}=\mspace{5mu} 0.32 \\

& P(x = information ) 

\mspace{5mu}=\mspace{5mu} \frac{6+4+1}{19} 

\mspace{5mu}=\mspace{5mu} \frac{11}{19} 

\mspace{5mu}=\mspace{5mu} 0.58 \\

& P(y = data ) 

\mspace{5mu}=\mspace{5mu} \frac{6+1}{19} 

\mspace{5mu}=\mspace{5mu} \frac{7}{19}  

\mspace{5mu}=\mspace{5mu} 0.37 \\[10pt]

& pmi(x = information,y = data)  \\

& \mspace{5mu}=\mspace{5mu} log\frac{P(x =information,y=data)}{P(x=information)\times P(y=data)} \\

&\mspace{5mu}=\mspace{5mu} log1.49 \\

&\mspace{5mu}=\mspace{5mu} 0.57  \\

\end{align}

$$


算出來的數字大於 0 , 表示 *information* 和 *data* 這兩個字的出現, 不是獨立事件



## 2.Implementation


我們用 `python nltk` 的 *brown corpus* 新聞類別文章, 來計算 *New* , *York* 的 *PMI* 和 *New* , *The* 的 *PMI* , 並比較兩者差異


將以下程式碼複製到 *pmi.py* 這個檔案



```python pmi.py

import nltk
from nltk.corpus import brown
from nltk import WordNetLemmatizer
from math import log 
wnl=WordNetLemmatizer()


_Fdist = nltk.FreqDist([wnl.lemmatize(w.lower()) for w in brown.words(categories='news')])

_Sents = [[wnl.lemmatize(j.lower()) for j in i] for i in brown.sents(categories='news')]

def p(x):
       return _Fdist[x]/float(len(_Fdist))

def pxy(x,y):
       return (len(filter(lambda s :  x in s and y in s ,_Sents))+1)/ float(len(_Sents) )

def pmi(x,y):
       return  log(pxy(x,y)/(p(x)*p(y)),2) 
       

```


其中 `_Fdist ` 是單字出現的頻率 , `_Sents` 是文章中所有的句子 , `p(x)` 計算單字 *x* 出現的機率,  ` pxy(x,y)` 計算單字 *x* 和單字 *y* 出現在同一個句子的機率, `pmi(x,y)` 計算單字 *x* 和單字 *y* 的 *Pointwise Mutual Information* 


到 *python* 的 *interactive mode* 載入這個檔案



```python

>>> from pmi import *

```


先來計算 *new* 和 *york* 各別出現的機率



```python

>>> p('new')
0.020265724857046755

>>> p('york')
0.004372687521022536

```


計算 *new* , *york* 出現在同一句子的機率



```python

>>> pxy('new','york')
0.011031797534068787

```


計算 *new* , *york* 的 *PMI*



```python

>>> pmi('new','york')
6.959890136179789

```


再來看看, *new* , *the* 出現在同一句子的機率



```python

>>> pxy('new','the')
0.04023361453601557

```


比 *new* , *york* 出現在同一句子的機率還高, 但是因為 *the* 出現次數較多

算一下 *the* 出現的機率



```python

>>> p('the')
0.5369996636394214

```


計算*new* , *the* 的 *PMI* , 還是沒有比 *new* , *york* 高



```python

>>> pmi('new','the')
1.8863664858873235

```


以上結果顯示, *PMI* 可以得出, *new* 和 *york* 一起出現的情形, 比較不是偶然的事件, 表示這兩個詞一起出現可能帶有較多訊息


雖然 *new* 和 *the* 出現的頻率比較多, 但同時 *the* 出現的頻率也比較多, 計算結果 *PMI* 可以把 *the* 的出現頻率除掉, 得出 *new* 和 *the* 的出現, 比較像是獨立事件


## 3.Reference


本文參考至coursera線上課程


#### D. Jurafsky, and C. Manning.  Natural Language Processing

https://www.coursera.org/course/nlp