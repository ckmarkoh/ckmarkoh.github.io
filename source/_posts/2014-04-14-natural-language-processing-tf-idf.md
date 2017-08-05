---
layout: post
title: 'TF-IDF'
date: 2014-04-14 10:12
comments: true
categories: ['Natural Language Processing','Text Mining', 'Information Retrieval']
---

## 1.Introduction


所謂的 *TF-IDF* , 是用來找出一篇文章中, 足以代表這篇文章的關鍵字的方法


例如, 有一篇新聞, 是 *nltk* 的 *Reuters Corpus* 中的文章, 這篇文章被歸類在 *grain* , *ship* 這兩種類別下, 文章的內容如下：



>GRAIN SHIPS LOADING AT PORTLAND 
>There were three grain ships loading and two ships were waiting to load at Portland , according to the Portland Merchants Exchange .


假設不知道什麼是 *TF-IDF*, 先用人工判別法試看看, 這篇新聞的關鍵字, 應該是 *Portland* , *ship* , *grain* 之類的字, 而不會是 *to* , *at* 這種常常出現的字

為什麼呢？因為 *to* 或 *at* 雖然在這篇文章中出現較多次, 但其他文章中也常有這些字, 所謂的關鍵的字, 應該是在這篇文章中出現較多次, 且在其他文章中比較少出現的字


所以,如果要在一篇文章中, 尋找這樣的關鍵字, 要考慮以下兩個要素:

<!--more-->

#### 1.這個字在這篇文章中出現的頻率( *Term-Frequency TF* )

#### 2.在所有的文章中,有幾篇文章有這個字( *Inverse-Document-Frequency IDF* )


這就是所謂的 TF-IDF


公式如下：

$$

TF\text{-}IDF

\mspace{5mu}=\mspace{5mu} TF \times IDF

\mspace{5mu}=\mspace{5mu} n_{w}^{d} \times log_{2} \frac{N}{N_{w}}

$$

其中, $$n_{w}^{d}$$ 表示在文章 $$d$$ 中, 文字 $$w$$ 出現的次數, $$N$$ 表示總共有幾篇文章, $$N_{w}$$ 表示有文字 $$w$$ 的文章有幾篇


例如以上 *Reuters Corpus* 文章的例子, *Portland* 在這篇文章中出現了 *3* 次, 所以 $$n_{w}^{d} =3 $$ , 而 *Reuters Corpus*  , 總共有 *10788* 篇文章, $$N = 10788$$ , 在這些文章中, 有 *Portland* 這個字的文章, 有 *28* 篇, 所以 $$N_{w} = 28.0$$ , 把這些數值帶入公式, 得出

$$

TF\text{-}IDF_{Portland} \mspace{5mu}=\mspace{5mu}  3 \times log_{2} \frac{10788}{28} \mspace{5mu}  \approx \mspace{5mu} 25.769

$$

如果是 *to* 這個字, 在這篇文章出現 *2* 次, 但是總共有 *6944* 篇文章有這個字, 所以算出來的 *TF-IDF*  是

$$

TF\text{-}IDF_{Portland} \mspace{5mu}=\mspace{5mu}  2 \times log_{2} \frac{10788}{6944} \mspace{5mu}  \approx \mspace{5mu} 1.271

$$


就這樣, 根據公式算出 *TF-IDF* 值的大小, 值越大的, 越可以作為代表這篇文章的關鍵字


## 2.Implementation


首先來瞭解一下 *nltk* 的 *Reuters Corpus*  和本文中的範例文章  

先到 *python* 的 *Interactive Mode* 載入 `reuters` 模組



```python

>>> from nltk.corpus import reuters

```


本文中的範例文章, 在 *Reuters Corpus*  裡面的 *Id* 為 `training/3386`

可以把文章內容印出來看看



```python

>>> fileid = 'training/3386'
>>> " ".join(reuters.words(fileids=fileid))
'GRAIN SHIPS LOADING AT PORTLAND There were three grain ships loading and two ships were \
waiting to load at Portland , according to the Portland Merchants Exchange .'

```


也可以看一下這篇文章的分類, 是被分在 *grain* 和 *ship*  這兩個類別



```python

>>> reuters.categories(fileids=fileid)
['grain', 'ship']

```


再來是實作 *TF_IDF* 的公式, 

開一個新的檔案, 命名為 *tf_idf.py* , 並貼上以下程式碼



```python tf_idf.py
from nltk.corpus import reuters
from math import log 
from nltk import WordNetLemmatizer
wnl=WordNetLemmatizer()

_DN = float(len(reuters.fileids()))
_RTS = {k:[wnl.lemmatize(w.lower()) for w in reuters.words(k)] for k in reuters.fileids()}

def idf(w):
    w = wnl.lemmatize(w.lower())
    return log( _DN / sum([float(w in _RTS[k]) for k in _RTS.keys() ]) , 2)

def tf(w,f):
    w = wnl.lemmatize(w.lower())
    return sum([float(w == x) for x in _RTS[f] ]) 

def tf_idf(w,f):
    return tf(w,f)*idf(w)

def run(f):
    word_tfidf = [(w,tf_idf(w,f)) for w in set(_RTS[f])] 
    for w,t in sorted(word_tfidf, key = lambda x : x[1], reverse=True):
        print "%-15s %.10f"%(w,t)
        

```


其中, `_DN` 是 *Reuters Corpus*  中, 所有文章的數量, 

而 `_RTS = {k:[wnl.lemmatize(w.lower()) for w in reuters.words(k)] for k in reuters.fileids()}` 

是先把所有文章中的字都轉成小寫, 並轉換成單數, 例如把 *SHIPS*, *ships* , *ship* 這三種字, 都轉成同一種字 *ship*

剩下的 *function*  `idf(w)` , `tf(w,f)` , `tf_idf(w,f)` 就是 *TF-IDF* 公式, 而 `run(f)` 是把一篇文章每一個字都去跑 *TF-IDF* , 並印出結果


再來到 *python* 的 *Interactive Mode* 載入 *tf_idf.py* 這個檔案



```python

>>> import tf_idf as ti

```


這可能需要花十幾秒甚至數分鐘的時間, 因為程式要把所有大小寫和單複數不同的字, 都轉成同樣的字, 要花一些時間


再來, 別忘了先前看的那篇文章 *id* 是什麼, 因為等一下還會用到



```python

>>> fileid = 'training/3386'

```


算一下 *Portland* 這個字在那篇文章中的 *TF* , *IDF* 以及 *TF-IDF*



```python

>>> ti.tf('Portland',fileid)
3.0

>>> ti.idf('Portland')
8.589784884178

>>> ti.tf_idf('Portland',fileid)
25.769354652534

```


還有, *to* 這個字在那篇文章中的 *TF* , *IDF* 以及 *TF-IDF*



```python

>>> ti.tf('to',fileid)
2.0

>>> ti.idf('to')
0.6355885737911242

>>> ti.tf_idf('to',fileid)
1.2711771475822484

```


再來,把文章中每個字的 *TF-IDF* 都算出來, 並做排序



```python

>>> ti.run(fileid)
portland        25.7693546525
ship            17.9126251546
loading         15.9417501031
grain           10.3270402590
load            8.7532836165
waiting         7.6967000881
merchant        7.4198598827
according       5.0979317878
were            4.9210037345
there           3.5817565104
exchange        3.4056179602
at              3.3358878942
three           3.0888007761
two             2.4827546741
to              1.2711771476
and             0.6732655874
the             0.6341349766
,               0.2614305201
.               0.1459533027

```


以上結果顯示, *Portland* , *ship* , *grain* 之類的關鍵字, 所算出來的 *TF-IDF* 值都比較高, 而 *to* , *at* , 或其他標點符號, 算出來的值就較低, 因此 *TF-IDF* 可以把文章中的關鍵字區分出來


## Reference 


本文使用 nltk 的 *Reuters Corpus* 做為範例, 若想知道 *Reuters Corpus* 要怎麼用, 請到以下網址

http://nltk.googlecode.com/svn/trunk/doc/book/ch02.html


另外, 本文的公式與觀念, 參考至以下這門 *coursera* 的線上課程 

#### Dr. Gautam Shroff.  Web Intelligence and Big Data

https://www.coursera.org/course/bigdata

