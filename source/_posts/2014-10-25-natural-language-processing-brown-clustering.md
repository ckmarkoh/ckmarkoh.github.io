---
layout: post
title: 'Brown Clustering'
date: 2014-10-25 13:31
comments: true
categories: ['Natural Language Processing','Text Mining','Machine Learning']
---

##Introduction


*Clustering* 是一種非監督式的機器學習方法。所謂非監督式的學習方法，即是不需要事先提供人工標記好的語料庫給機器學習的演算法。可以直接從未標示的語料庫中，根據既有的特徵來做分類。


例如，可以根據某字的前面或後面有哪些字，來決定哪些字屬於同一類。給一語料庫如下：



>

The dog runs.

A dog jumps.

The dog jumps.

A cat runs.

The cat jumps.

The cat runs.


>


根據這些例句，可以把 *cat* 和 *dog* 歸在同一類，因為它們前面的字是 *the* 或 *a* ，同理，也可以把 *run* 和 *jump* 歸在同一類。


## Defining the Formulation


現在來定義一下，進行這種分類所需要的數學公式。

<!--more-->


假設總共有 $$V$$ 個字彙， $$V=\{w_{1},w_{2},...,w_{t}\}$$ ，和一個分類函數 $$C:V \rightarrow \{1,2,...k\}$$ 。 $$C$$ 把 $$V$$ 中的單字分類成 $$k$$ 類， $$k \leq t$$ 。

例如， $$w_{1}$$ 被分類到類別 $$3$$ ，則 $$C(w_{1}) = 3$$ 。


給定語料庫中的句子 $$S = w_{1}, w_{2}, ... ,w_{n} $$ ，則可以計算這個句子出現的機率，為：


$$

p(w_{1},w_{2},...,w_{n}) = \prod_{i=1}^{n}  e(w_{i}\mid C(w_{i})) \times q(C(w_{i}) \mid C(w_{i-1}))


$$


其中，$$e(w_{i}\mid C(w_{i}))$$  為，在類別 $$ C(w_{i}) $$ 中，出現 $$w_{i}$$ 的機率。 

而 $$q(C(w_{i}) \mid C(w_{i-1}))$$ 為，若此字的類別為 $$ C(w_{i}) $$ ，則前一個字的類別為 $$C(w_{i-1})$$ 的機率。


根據先前例子中的語料庫，假設已經分類完成，分類 $$(1)$$ 如下：


$$

\begin{align}

&  C(\text{the})=C(\text{a})=1, \mspace{10mu}

 C(\text{dog})=C(\text{cat})=2, \mspace{10mu}

 C(\text{run})=C(\text{jump})=3 \mspace{10mu} & (1)

\end{align}

$$


並且，可計算出 $$e(w_{i}\mid C(w_{i}))$$ 和 $$q(C(w_{i}) \mid C(w_{i-1}))$$ 的機率。 例如， $$e(\text{the}\mid C(\text{the}) )$$ 的機率為：


$$

e(\text{the}\mid C(\text{the}) )= 

e(\text{the}\mid 1 )= 

\frac{count(\text{the})}{count(\text{the})+count(\text{a})} = 

\frac{4}{4+2} = \frac{2}{3}

$$


其中，$$count(\text{the})$$ 為 *the* 在語料庫中出現的次數。 而 $$q(C(\text{dog}) \mid C(\text{the}))$$ 的機率為：


$$


q(C(\text{dog}) \mid C(\text{the})) = q(2 \mid 1) = \frac{count(1, 2 )}{count(1)} = \frac{6}{6} = 1


$$


其中，$$count(1,2)$$ 為，類別為 $$2$$ 的字，出現在類別為 $$1$$ 的字後面的機會。


給定語料庫中的句子 *The dog runs* ，那麼可以算出此句的機率為：


$$

\begin{align}

& p(\text{the}, \text{dog}, \text{run}) \\

& = e(the \mid C(\text{the})) \times e(dog \mid C(\text{dog})) \times e(run \mid C(\text{run})) \\ 

& \times q(C(\text{the}) \mid 0 ) \times q(C(\text{dog}) \mid C(\text{the}) )  \times q(C(\text{run}) \mid C(\text{dog}) ) \\

& = e(the \mid 1) \times e(dog \mid 2) \times e(run \mid 3)  \times q(1 \mid 0 ) \times q(2 \mid 1 )  \times q( 3 \mid 2 ) \\

& = \frac{2}{3} \times \frac{1}{2} \times \frac{1}{2}  \times 1  \times 1  \times 1  \\

& = \frac{1}{6}

\end{align} 

$$

其中，把第一個字 *the* 的前一個字，歸類為第 $$0$$ 類，即可求出 $$q(C(\text{the}) \mid 0 ) = 1$$ 。


再給一個分類，分類 $$(2)$$，這次隨便分，把 *the* 和 *dog* 分成一類，把 *a* 和 *cat* 分成一類，再把 *run* 和 *jump* 分成一類，分類如下：


$$

\begin{align}

& C(\text{the})=C(\text{dog})=1, \mspace{10mu}

 C(\text{a})=C(\text{cat})=2, \mspace{10mu}

 C(\text{run})=C(\text{jump})=3 \mspace{10mu} & (2)

\end{align}

$$


給定語料庫中的句子 *The dog runs* ，那麼可以算出此句的機率為：


$$

\begin{align}

& p(\text{the}, \text{dog}, \text{run}) \\

& = e(the \mid C(\text{the})) \times e(dog \mid C(\text{dog})) \times e(run \mid C(\text{run})) \\ 

& \times q(C(\text{the}) \mid 0 ) \times q(C(\text{dog}) \mid C(\text{the}) )  \times q(C(\text{run}) \mid C(\text{dog}) ) \\

& = e(the \mid 1) \times e(dog \mid 1) \times e(run \mid 3)  \times q(1 \mid 0 ) \times q(1 \mid 1 )  \times q( 3 \mid 1 ) \\

& = \frac{4}{7} \times \frac{3}{7} \times \frac{1}{2}  \times \frac{4}{6}  \times \frac{2}{7}  \times \frac{1}{2}  \\

& = \frac{4}{343}

\end{align} 

$$


從以上例子得知，用語料庫的句子  *The dog runs* 來算機率，分類 $$(1)$$ 得出的機率比分類 $$(2)$$ 高。若要得到較理想的分類，可以用語料庫的句子算出的機率，做最佳化，機率較高者，為較理想的分類。



至於，把語料庫中 $$t$$ 個單字 分組成 $$k$$ 個類別的過程如何？

首先，把 $$t$$ 個單字個別分成 $$t$$ 組。

再從這些組中挑兩組合併起來，使其得出機率為最大值。合併完後，共有 $$t-1$$ 組。

再來，從這些 $$t-1$$ 組中，再挑兩組合併，以此類推，直到剩下 $$k$$ 組。


由於此種演算法的時間複雜度相當高，為$$O(t^{5})$$ ，*[Brown et al., 1992]* 提出的 *hierarchical clustering* 的概念，可有效降低時間複雜度，有興趣者請看延伸閱讀。



## Implementation


首先，建立一個程式檔，命名為 *bcluster.py*



```python bcluster.py
W_CORPUS = [
  ['the','dog','run'], ['a','dog','jump'],
  ['the','dog','jump'], ['a','cat','run'],
  ['the','cat','jump'], ['the','cat','run'],
]

def word_count(c):
  wcount = {} 
  for s in c:
    for w in s:
      wcount.update({w:wcount.get(w,0) + 1.0})
  return wcount

def choose_merge(v, w_corpus, w_count):
  max_p, max_v = 0.0, []
  for i in range(len(v)):
    for j in range(i+1,len(v)):
      s1 = [x for x in v]
      s2 = [ s1.pop(i)+s1.pop(j-1) ]+[x for x in s1]
      p = calculate_prob(s2, w_corpus, w_count)
      if p > max_p:
        max_p, max_v = p, s2 
  return max_v           

def calculate_prob(v, w_corpus, w_count):
  w_class = { w:str(i) for i,s1 in enumerate(['0']+v) for w in s1}
  c_corpus = [[w_class.get(w) for w in s] for s in w_corpus]  
  c_count = word_count(c_corpus)
  gran_count = word_count([["_".join(s[i:i+2]) for i in range(len(s))] for s in c_corpus])
  p = 1.0;
  for s in w_corpus:
    for i in range(1,len(s)):
      p = p*e(s[i], w_class, w_count, c_count)*q(s[i], s[i-1], w_class, c_count, gran_count)
  return p 

def e(w, w_class, w_count, c_count):
  return w_count[w] / c_count[w_class[w]] 

def q(w, wp, w_class, c_count, gran_count):
  return gran_count["%s_%s"%(w_class[wp],w_class[w])] / c_count[w_class[w]]
    
def bcluster(k, corpus):
  w_count = word_count(corpus)
  w_corpus = [ ['0']+ s for s in corpus]
  v = [[w] for w in w_count.keys()]
  while len(v) > k:
  	print v
    v = choose_merge(v, w_corpus, w_count)
  return v

```


其中，`W_CORPUS` 為語料庫，`bcluster(k, corpus)` 為主要執行 *cluster* 的函數，參數 `k` 為總共要分成幾組。 


到 python interactive mode 載入模組



```python

>>> from bcluster import bcluster,W_CORPUS

```


根據語料庫中的文字，分成三組，程式印出逐漸將6個字分成3組的過程，最後一行為最終結果：



```python

>>> from bcluster import bcluster,W_CORPUS
>>> bcluster(3,W_CORPUS)
[['a'], ['jump'], ['run'], ['the'], ['dog'], ['cat']]
[['a', 'the'], ['jump'], ['run'], ['dog'], ['cat']]
[['dog', 'cat'], ['a', 'the'], ['jump'], ['run']]
[['jump', 'run'], ['dog', 'cat'], ['a', 'the']]

```


## Reference


本文參考至coursera線上課程

####  Michael Collins. Natural Language Processing

https://www.coursera.org/course/nlangp


Brown Clustering 出處：

#### Brown et al. Class-Based n-gram Models of Natural Language, 1992
