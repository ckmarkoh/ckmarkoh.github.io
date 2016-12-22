---
layout: post
title: 'Ngram Smoothing / Interpolation'
date: 2014-03-28 11:57
comments: true
categories: ['Natural Language Processing']
---

以下為我對於 *Ngram Smoothing* 這個章節的公式所做的筆記


## Introduction


*Ngram Smoothing* 是用於出現在 *Training Corpus* 中沒有的 *Ngram*,

它的機率會是0

本文以 *Bigram* 為例, *Bigram* 的機率如下:


$$

P(w_{n}\mid w_{n-1} )=  \frac{C(w_{n-1},w_{n})}{C(w_{n-1})}

$$


如果 $$C(w_{n-1},w_{n})=0$$ , 則 $$P(w_{n}\mid w_{n-1} )= 0$$

為了避免probability為0, 我們要做 Ngram Smoothing

<!--more-->

## Laplace Smoothing


最簡單的想法, 就是把每一個 *Ngram* 的 *count* 都加1, 然後再做 *Normalization*


$$

c^{*}_{i}=(c_{i}+1) \frac{N}{N+V} \\

P^{*}_{lablace}=\frac{(c_{i}+1) }{N+V}


$$


其中, $$V$$ 是所有 *Ngram* 的種類


若是用於 *bigram* , 則機率為：


$$

P^{*}_{lablace}(w_{n}\mid w_{n-1} )=  \frac{C(w_{n-1},w_{n})+1}{C(w_{n-1})+V}

$$



## Good-Turing Discounting


這種方法是由 *Good (1953)* 提出

是根據 *Ngram* 的 *count* 來估計, 例如 *count* 為 *n* 的 *Ngram*, 用 *Nount* 為 *n+1* 的 *Ngram* 來估計：


$$

c^{*}=(c+1) \frac{N_{c+1}}{N_{c}}

$$


其中 $$N_{c}$$ 為 *count* 是 *c* 的 *Ngram* 個數


例如, *count* 為 *0* 的 *Ngram* ,個數為

$$

c^{*}=(0+1) \frac{N_{0+1}}{N_{0}}=\frac{N_{1}}{N_{0}}

$$


若當我們要計算出現 $$N_{0}$$ , *(things with frequency zero in training)* , 的機率,

可設 $$N_{0}=1$$ ,則：

$$

P^{*}_{GT}=\frac{\frac{N_{1}}{N_{0}}}{N}=\frac{N_{1}}{N}

$$


舉個例子,河裡面有八種魚,釣到的次數如下：


| carp | perch | whitefish | trout | salmon | eel | catfish | bass |
|----- |------ |-----------|-------|--------|-----|---------|------|
| 10   | 3     | 2         | 1     | 1      | 1   | 0       | 0    |       


則下一次釣到出現 *未出現過的魚 （catfish or bass)* 的機率為：

$$

P^{*}_{GT}(unseen)=\frac{3}{18}=0.17

$$


若已知道未出現過的魚有兩種, 則釣到 *catfish* 的機率為：

$$

P^{*}_{GT}(catfish)=\frac{0.17}{2}=0.085

$$


再來看看釣到 *trout* 的機率：

$$

c^{*}(trout)=2\times\frac{N_{2}}{N_{1}}=2\times\frac{1}{3}=0.67 \\

P^{*}_{GT}(unseen)=\frac{0.67}{18}=0.037

$$


## Advanced Good-Turing

因為可能當 $$c>0$$ 時, $$N_{c}=0$$

這種情況, 用 *linear regression* 在log space去smooth $$N_{c}$$,如下：

$$

\log(N_{c})=a+b\log(c)

$$

再來, 於 *Katz, S. M. (1987)* 提到

*discounted estimation* 並不適用於所有的 $$c^{*}$$, 

$$

c^{*}=


\begin{cases} 

c &\text{  for  } \mspace{10mu} c>k \\[14pt]


\frac{

(c+1)\frac{N_{c+1}}{N_{c}}-c\frac{(k+1)N_{k+1}}{N_{1}}

}{

1-\frac{(k+1)N_{k+1}}{N_{1}}

}

 &\text{  for  } \mspace{10mu} 1\leq c\leq k

 \end{cases}

 

$$

## Interpolation:


因為 *Trigram* 的 *count* 可能為0,

我們可以用內插法把 *Trigram* 的 *Count* 表示成 *Trigram*, *Bigram* 和 *Unigram* 的內插

如下：


$$

\overset{\wedge}{P} = \lambda _{1} P(w_{n}\mid w_{n-1}w_{n-2})+\lambda_{2} P(w_{n} \mid w_{n-1}) + \lambda_{3}P({w_{n}}) \\[14pt]

\sum\limits_{i} \lambda_{i} = 1

$$


我們也可以讓 $$\lambda$$ 可跟著前後文調整數值,像這樣：


$$

\overset{\wedge}{P} = \lambda _{1}(w^{n-1}_{n-2}) P(w_{n}\mid w_{n-1}w_{n-2})+\lambda_{2}(w^{n-1}_{n-2}) P(w_{n} \mid w_{n-1}) + \lambda_{3}(w^{n-1}_{n-2})P({w_{n}}) 


$$


至於 $$\lambda$$ 的值要怎麼算呢？可以用 *Maximum Likelihood Estimation* 求得



## Reference:


本篇擷取自教科書：

Daniel Jurafsky & James H. Martin.Speech and Language Processing: An introduction to natural language processing,computational linguistics, and speech recognition. 


其他參考文獻：

Good, I. J. (1953). The population frequencies of species and the estimation of population parameters. Biometrika, 40,16–264.

Katz, S. M. (1987). Estimation of probabilities from sparse data for the language model component of a speech recogniser
