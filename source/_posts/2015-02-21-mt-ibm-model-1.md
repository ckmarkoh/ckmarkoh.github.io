---
layout: post
title: '機器翻譯 -- IBM Model 1'
date: 2015-02-21 09:31
comments: true
categories: 
---

## Introduction


繼續之前[統計機器翻譯](/blog/http://cpmarkchang.logdown.com/posts/239855-machine-translation-statistical-machine-translation)所提到的，要計算 *Translation Model* $$p(F∣E)$$ ，即給定某個英文句子 $$E$$ ，則它被翻自成中文句子 $$F$$ 的機率是多少？


計算 *Translation Model* 需要用到較複雜的模型，例如 *IBM Model* 。而 *IBM Model 1*  是最基本的一種 *IBM Model* 。


## Alignment


在講 *IBM Model 1* 之前，要先介紹 *alignment* 是什麼。因為，要將英文翻譯成中文，則中文字詞的順序可能跟英文字詞的順序，不太一樣。例如：



>這不是一個蘋果
>This is not an apple 


這兩個句子，在中文中的 **"不是"** 英文為 **"is not（是不）"** 。為了考慮到字詞順序會變，因此要定義 *alignment* ，來記錄中文句子中的哪個字，對應到英文句子中的哪個字。


![](/images/pic/pic_00075.tiff)


此例中， *alignment* 為 $$\{1\rightarrow 1,3\rightarrow 2,2\rightarrow 3,4 \rightarrow 4,5\rightarrow 5 \}$$，表示第一個中文字對應到第一個英文字，第二個中文字對應到第三個英文字，以此類推。

<!--more-->


## IBM Model 1


*IBM Model 1* 的公式如下：


$$

P(F\mid E) = \sum_{A} P(F,A \mid E) = \sum_{A} P(F \mid E, A) \times P(A \mid E)

$$


此公式可分為 $$P(F \mid E, A)$$ 和 $$P(A \mid E)$$ 兩部分。


第一部分為 $$P(F \mid E, A)$$ ，給定一個 *alignment* $$A$$ ，和英文句子 $$E = (e_{1},e_{2}...,e_{I})$$ ，定義中文句子 $$F = (f_{1},f_{2},...,f_{J})$$ 的出現機率為：


$$

  P(F \mid E, A) = \prod_{j=1}^{J} t(f_{j} \mid e_{A_{j}})


$$


其中，$$ t(f_{j} \mid e_{A_{j}}) $$ 表示在 *alignment* $$A$$ 之下，對應到中文字 $$f_{j}$$的英文字 $$e_{A_{j}}$$ ，翻譯成中文字 $$f_{j}$$ 的機率。

舉例，用先前提到的例句，以此公式算出的機率為：


$$

  P(F \mid E, A) =  

  t(\text{這} \mid \text{this}) \times

  t(\text{不} \mid \text{not}) \times

  t(\text{是} \mid \text{is}) \times

  t(\text{一個} \mid \text{an}) \times

  t(\text{蘋果} \mid \text{apple}) 

$$


其中， $$t(\text{這} \mid \text{this})$$ 表示英文字 **"this"** 翻譯成中文字 **"這"** 的機率。 


第二部分為$$P(A \mid E)$$，給定長度為 $$I$$ 的英文句 $$E$$，將翻譯成長度為 $$J$$ 的中文句，則 *alignment* 為 $$A$$ 的機率為：


$$

  P(A \mid E) = \frac{\epsilon}{(I+1)^{J}}


$$


 *IBM Model 1* 採用最簡單的假設：**假設每個alignment所發生的機率都相同** 。如果中文字長度為 $$J$$ ，英文長度為 $$I$$ ，則共有 $$I^{J}$$ 種組合。但實際上，有些中文字（例如：的），沒有相對應的英文字，所以共有 $$(I+1)^{J}$$ 種可能。另外，公式中的 $$\epsilon$$ 為一常數。


綜合以上兩項，可以得出 *IBM Model 1* 的公式為：


$$

\begin{equation}

 P(F\mid E) = \sum_{A} P(F,A \mid E) = \sum_{A} P(F \mid E, A) \times P(A \mid E) \\

 = \sum_{A}  \prod_{j=1}^{J} t(f_{j} \mid e_{A_{j}}) \times  \frac{\epsilon}{(I+1)^{J}} \\

\end{equation}

$$


通常，在求解的時候，不會把每一種 *alignment* 的結果都加起來，而是求出機率最大的 *alignment* ，並採用它來計算 *IBM Model* 的值，如下：

 

$$

\mathop{\arg\,\max}\limits_{A} P(F,A \mid E)

= \mathop{\arg\,\max}\limits_{A} \prod_{j=1}^{J} t(f_{j} \mid e_{A_{j}}) \times  \frac{\epsilon}{(I+1)^{J}} 


$$


## EM algorithm


在進行[統計機器翻譯](/blog/http://cpmarkchang.logdown.com/posts/239855-machine-translation-statistical-machine-translation)的演算法時，需從平行語料庫中訓練出模型。但通常平行語料庫中不會標記 *alignment* ，可是必須要先有 *alignment* ，才能得出 $$t(f_{j} \mid e_{A_{j}})$$ 的值，這樣才能用 *IBM Model 1* 來計算。


這時就需要用到 *EM algorithm* 從平行語料庫中，計算出 $$t(f_{j} \mid e_{A_{j}})$$ 的值，以得出機率最大的 *alignment*。


設 $$(F^{(k)}, E^{(k)})$$ 為共 $$k$$ 個句子的 *中-英* 平行語料庫，其中，$$ F^{(k)} = (f_{1}^{(k)}, f_{2}^{(k)},...,f_{m_{k}}^{(k)} )$$ 為第 $$k$$ 個中文句子， $$f_{1}^{(k)}$$ 為此句第一個中文字，  $$m_{k}$$ 為句子長度。$$ E^{(k)} = (e_{1}^{(k)}, e_{2}^{(k)},...,e_{l_{k}}^{(k)} )$$ 為第 $$k$$ 個英文句子。演算法如下：


$$

\begin{align}

\text{1} & \text{For all e , f, set } t(f\mid e)  = 1 \mathbin{/} V \\

\text{2} & \textbf{for } s = 1 ... S \\ 

\text{3} & \mspace{20mu} \text{Set all counts } c = 0 \\

\text{4}& \mspace{20mu} \textbf{for } k = 1...N \\

\text{5}& \mspace{40mu} \textbf{for } i = 1...m_{k} \\

\text{6}& \mspace{60mu} \textbf{for } j = 0...l_{k} \\

\text{7}& \mspace{80mu} \delta (k,i,j) = t( f_{i}^{(k)} \mid e_{j}^{(k)}) \mathbin{/}  \Sigma_{j=0}^{l_{k}} t( f_{i}^{(k)} \mid e_{j}^{(k)}) \\

\text{8}& \mspace{80mu} c(e_{j}^{(k)}, f_{i}^{(k)}) \leftarrow c(e_{j}^{(k)}, f_{i}^{(k)})  + \delta (k,i,j) \\

\text{9}& \mspace{80mu} c(e_{j}^{(k)}) \leftarrow c(e_{j}^{(k)})  + \delta (k,i,j) \\

\text{10}& \mspace{20mu} \text{Set all } t(f\mid e) = c(e,f) \mathbin{/}  c(e) \\

\text{11}& \textbf{return } t


\end{align}

$$

第1行先將所有的$$t(f \mid e)$$ 初始化為 $$\frac{1}{V}$$ ，其中，$$V$$ 為語料庫中的中文詞類數量。

第2行的 $$S$$ 表示，總共進行了 $$S$$ 次的運算。

第3行的 $$c$$ 用來計 $$t(f \mid e)$$ 累加的值，初始化先設定為 $$0$$ 。

第4行用迴圈跑平行語料庫中所有的句子

第5到第9行迴圈用來逐一跑中文和英文句中的每一個字。

第5行跑中文句中的中文字。

第6行跑英文字。由於有些中文字，沒有相對應的英文字，故英文句的迴圈從 *0* （沒有對應到任何字）開始跑。

第7行算出 $$\delta (k,i,j)$$ 的值，並於第8,9行累加到 $$c$$ 中。

第10行，算完所有的句子之後，用 $$c(e,f)$$ 和 $$c(e)$$ 來更新 $$t(f\mid e)$$ 的值，再回到第3行，用新的 $$t$$ 值來計算新的 $$c$$ 值。

第11行，以上迴圈重複跑過 $$S$$ 次以後，回傳 $$t$$ 的值。


## Implementation


給定以下的平行語料庫



> 一本書 : a book
> 一本雜誌 : a magazine
> 這本書 : this book
> 這本雜誌 : this magazine 


可用以下的程式碼 *EM Algorithm* 算出 $$t$$ 的值。



```python ibm1.py
#-*- encoding:utf-8 -*-
import operator
CORPUS_CH = [ ['一本','書'], ['一本','雜誌'], ['這本','書'], ['這本','雜誌'], ]
CORPUS_EN = [ ['a','book'], ['a','magazine'], ['this','book'], ['this','magazine'], ]
f_word = list(set(reduce(operator.add,CORPUS_CH)))
e_word = list(set(reduce(operator.add,CORPUS_EN)))
T = {}

for k in range(5) :
  C = {}
  for m,l in zip(CORPUS_CH,CORPUS_EN):
    if k == 0:
      for fi in m:
        for ej in l:
          if "%s|%s"%(fi,ej) not in T:
            T["%s|%s"%(fi,ej)]  =  1.0/len(e_word)
    for i, fi in enumerate(m):  
      sum_t = sum([ T["%s|%s"%(fi,ej)] for ej in l]) * 1.0
      for j, ej in enumerate(l):
        delta = T["%s|%s"%(fi,ej)] / sum_t
        C["%s %s"%(ej, fi)] =  C.get("%s %s"%(ej, fi), 0) + delta
        C["%s"%(ej)] = C.get("%s"%(ej), 0) + delta
  print "---iteration:%s---"%(k) 
  for key in T:
    print key,":",T[key]
  for f in f_word:
    for e in e_word:
      if "%s %s"%(e, f) in C and "%s"%(e) in C:
        T["%s|%s"%(f,e)] = C["%s %s"%(e, f)] / C["%s"%(e)]
        
print "---iteration:%s---"%(k+1)
for key in T:
  print key,":",T[key]


```


執行 *python ibm1.py* 計算結果：



```bash

> python ibm1.py 
---iteration:0---
雜誌|magazine : 0.25
書|this : 0.25
這本|book : 0.25
這本|this : 0.25
一本|magazine : 0.25
雜誌|a : 0.25
雜誌|this : 0.25
書|book : 0.25
一本|a : 0.25
這本|magazine : 0.25
一本|book : 0.25
書|a : 0.25
...
---iteration:5---
雜誌|magazine : 0.941176470588
書|this : 0.0294117647059
這本|book : 0.0294117647059
這本|this : 0.941176470588
一本|magazine : 0.0294117647059
雜誌|a : 0.0294117647059
雜誌|this : 0.0294117647059
書|book : 0.941176470588
一本|a : 0.941176470588
這本|magazine : 0.0294117647059
一本|book : 0.0294117647059
書|a : 0.0294117647059

```


其中，初始值的 $$t$$ ，不管是哪個中文字對應到哪個英文字，都是 *0.25* 。經過了五次的迴圈運算後，  $$t$$ 值會逐漸收斂，漸與訓練資料吻合。例如 $$t(\text{雜誌} \mid \text{magazine}) = 0.941$$ ，大於 $$t(\text{一本} \mid \text{magazine}) = 0.029$$ 。表示英文字 *magazine* 較可能翻譯成中文字詞 **"雜誌"** ，而不是 **"一本"** 。


## Reference


本文參考至coursera線上課程

####  Michael Collins. Natural Language Processing

https://www.coursera.org/course/nlangp
