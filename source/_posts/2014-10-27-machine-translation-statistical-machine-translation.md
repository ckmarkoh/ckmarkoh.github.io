---
layout: post
title: '機器翻譯 -- Statistical Machine Translation'
date: 2014-10-27 16:49
comments: true
categories: 
---

## Introduction


*Statistical Machine Translation* （統計機器翻譯）是一種機器翻譯的演算法，這種方法藉由從 *parallel corpus*（平行語料庫）語料庫，當成訓練資料，訓練出機器學習的模型，以此將句子翻譯成另一個句子。


平行語料庫中包含大量句子，這些句子意思一樣，但分別用兩種語言寫成，例如：



```
這是一個蘋果。
This is an apple.

桌上有一本書。
There is a book on the table．

...... 

```


藉由這種平行語料庫，就可以用統計的方式，讓機器學會如何將一種語言，翻譯成另一種語言。


## Noisy Channel Model


*Noicy Channel Model* 是一種常用的統計機器翻譯的模型。

<!--more-->


如果，要把中文翻譯成英文，則可用計算某個中文句子翻譯成英文句子的機率，來找出哪個翻譯結果，是比較好的翻譯。給定中文句子 *c* ，它翻譯成某個英文句子 *e* 的機率，為 $$p(e \mid c)$$ ，即為，在中文句子為 *c* 的條件下，出現英文句子 *e* 的條件機率。


在 *Noicy Channel Model* 中，有兩個成分：


$$

\begin{align}

& p(e)  & \text{The Language Model}   \\

& p(c \mid e)  & \text{The Translation Model}  

\end{align}

$$


其中， *Language Model* 表示英文句子 *e* 在語料庫中的機率。要建立此模型，用單一一種語言的語料庫做訓練即可。

而 *Translation Model* 表示給定英文句子 *e* ，在此條件下，翻譯成中文句子 *c* 的機率。要建立此模型，需要用平行語料庫做訓練。


用貝氏定理，得出某個中文句子 *c* 翻譯成某個英文句子 *e* 的機率，為：


$$

p(e \mid c) = \frac{p(e,c)}{p(c)} = \frac{p(e)\times p(c \mid e)}{\sum_{e} p(e)\times p(c \mid e)}

$$


若要得出翻譯最佳的英文句子，則可以求機率最大者，如下：


$$

\mathop{\arg\,\max}\limits_{e}  p(e \mid c) = \mathop{\arg\,\max}\limits_{e} p(e)\times p(c \mid e)

$$



## Example


舉個例子，若要將中文句 *「我肚子餓了」* 翻譯成英文。假設已經先從從語料庫中，計算出可能翻譯出的英文句子，如下：


$$

\begin{array}{c | c c}

English & p(c \mid e) & p (e) \\ \hline

\text{I am hungry} & 0.00019 & 0.0084\\

\text{My belly hungry} & 0.00031 & 0.0000031\\

\text{I starve} &  0.00045 & 0.0000012\\

\end{array}

$$


則這句中文最有可能的翻譯為 *"I am hungry"* ：


$$

\begin{align}

& \mathop{\arg\,\max}\limits_{e}  p(e \mid c) \\

& = \mathop{\arg\,\max}\limits_{e} p(e)\times p(c \mid e) \\

& = p(e = \text{"I am hungry"} )\times p(c \mid e = \text{"I am hungry"} ) \\

& =  0.00019 \times 0.0084 = 0.000001596 

\end{align}

$$


註：本例子已先假設 *Language Model* $$p(e)$$ 和 *Translation Model* $$p(c \mid e)$$ 都已經先計算好。若要從頭計算 *Language Model* 可以用 *n-gram* ，但計算 *Translation Model* 則需要更複雜的模型，例如 *IBM Model* 或 *Phrase-based Model* 。


## Reference


本文參考至coursera線上課程

####  Michael Collins. Natural Language Processing

https://www.coursera.org/course/nlangp
