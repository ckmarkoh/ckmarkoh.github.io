---
layout: post
title: 'Naive Bayes Classifier'
date: 2014-04-13 07:04
comments: true
categories: ['Machine Learning', 'Natural Language Processing']
---

## 1.Introduction


在自然語言處理的應用, 常常有分類的問題, 例如把某篇新聞分到哪一類

處理分類問題, 有種簡單的方法, 就是看這篇文章有哪些關鍵字, 根據這些關鍵字的出現與否, 用 *Naive Bayes Classifier* 做分類


要講 *Naive Bayes Classifier* 之前, 首先, 要知道 *Bayes rule* 是什麼, *Bayes rule* 很簡單, 如下

$$

P(A,B)=P(A\mid B) \times P(B) = P(B\mid A) \times P(A)

$$

這個公式, 高中數學應該都有教過 , 如果 $$A$$ 和 $$B$$ 為 *Independence* , 則

$$

\begin{align}

&P(A\mid B) = P(A) \\

&P(A,B)=P(A\mid B) \times P(B) = P(A) \times P(B)

\end{align}

$$


所謂的 *Naive Bayes Classifier*  ,  其實就是應用 *Bayes rule*  來處理分類問題

<!--more-->

舉個例子, 文章分類問題, 如果有一篇文章有關鍵字有 $$W_{1} , W_{2}$$ ,假設 $$P(W_{1})$$  和 $$P(W_{2})$$ 為 *Independence* , 這篇文章被分到類別 $$C$$ 的機率

$$

\begin{align}

&P(C,W_{1},W_{2}) \\

&=P(C \mid W_{1},W_{2}) \times P(W_{1},W_{2})  &(Bayes Rule) \\

&= P(W_{1},W_{2}\mid C ) \times P(c) &(Bayes Rule) \\ 

&= P(W_{1}\mid W_{2}, C) \times P(W_{2} \mid C)  \times P(c ) &(Bayes Rule) \\

&= P(W_{1}\mid C ) \times P(W_{2} \mid C )  \times P(C )  & (Independence)

\end{align}

$$


如果某篇文章有關建字 $$W_{1}$$ , 但沒有關鍵字 $$W_{2}$$ , 這篇文章被分到類別 $$C$$ 的機率,如下


$$

\begin{align}

&P(C , W_{1} ,  W_{2}')  \\

&= P(W_{1}  \mid C ) \times P( W_{2}'  \mid C)  \times P( C) \\

&= P(W_{1}  \mid C ) \times (1- P( W_{2}  \mid C))  \times P( C)

\end{align}

$$


其中, 沒有關鍵字 $$W_{2}$$ 字的機率, 取有關鍵字 $$W_{2}$$ 的機率的補集,  $$P( W_{2}'  \mid C)= 1- P( W_{2}  \mid C )$$


如果所挑選的關鍵字超過兩個以上, 某篇文章有關鍵字 $$W_{1} , W_{2},...,W_{k}$$ , 但沒有關鍵字 $$V_{1},V_{2},...,V_{l}  $$ 則此文章被分到類別 $$C$$ 的機率如下

$$

\begin{align}

&P(C,W_{1},W_{2},...,W_{k}, V_{1}' , V_{2}' , ..., V_{l}' ) \\

&= \prod_{i=1}^{k} P(W_{i}\mid C ) \times \prod_{j=1}^{l} (1-P(V_{j}\mid C )) \times P(C )  

\end{align}

$$


來分析一下以上公式的物理意義


| 成份 | 物理意義   |
|:--------------------|:----------------------------------------------------------------------------|
|  $$P(C )$$           |  在所有的文章中,  類別 $$C$$ 所占的比例, 若比例越高, 就表示文章越有可能屬於此類別 | 
|  $$P(W_{i}| C )$$ |  若某文章的類別是 $$C$$ , 則此文章有關鍵字 $$W_{i}$$ 的機率              | 
|  $$1- P(W_{i}| C )$$ |  若某文章的類別是 $$C$$ , 則此文章沒有關鍵字 $$W_{i}$$ 的機率         | 


所以, 把以上三種全部乘起來, 得到文章和關鍵字的 *Joint Probablity* , 就可以用關鍵字來預測文章的類別


## 2.Example : Sentiment Analysis


*Sentiment Analysis* 是一種分類問題, 就是把評論性質的文句, 分類到 *正面評論* 和 *負面評論* 這兩類


例如, 在課程的討論版上, 蒐集各種關於某門課的評論句子, 句子的種類如下


| count | sentence                                                                 |sentiment|
|:------|:-------------------------------------------------------------------------|---------|
| 2000  | I really **like** this course and I learn a **lot** from it              |  +      |
| 800   | I **hate** this course and I think it is a **waste** of time             |  -      | 
| 200   | The course is extremely **simple** and quite a **bore**                  |  -      | 
| 3000  | The course is **simple**, and very **easy** to follow                    |  +      | 
| 1000  | I **enjoy** this course a **lot** and learning something too             |  +      | 
| 400   | I would **enjoy** myself a **lot** if i did not have to take this course |  -      | 
| 600   | I didn't **enjoy** this course                                           |  -      | 


其中, 欄位 *count* 表示這種句子出現的次數, 句子中的粗體字的為 *keyword* , 欄位 *sentiment* 表示正評或負評, 正評記為 *+* , 負評記為 *-* , 給一個新的評論句子 


$$

 \text{I really } \textbf{like } \text{this } \textbf{simple } \text{course a } \textbf{lot } \text{.}

$$


這個句子中有關鍵字 **like** , **simple** , **lot** , 但沒有關鍵字 **hate** , **waste** , **bore** , **easy** , **enjoy** , 則這個句子為正面評論的機率為

$$

\begin{align}

&P(+,like,simple,lot,hate',waste',bore',easy',enjoy') \\

&=P(+) \times P(like \mid  +) \times P(simple \mid  +) \times P(lot \mid  +)  \times (1- P(hate \mid  +)) \\

& \times (1- P(waste \mid  +)) \times (1- P(bore \mid  +)) \times (1- P(easy \mid  +)) \times (1- P(enjoy \mid  +))  

\end{align}

$$

先分別算出這些項目的機率, 再全部乘起來

$$

\begin{align}

& P(+) =	\frac{2000+3000+1000}{2000+800+200+300+1000+4000+600} = \frac{6000}{8000} = 0.75 \\[10pt]

& P(like \mid  +) =	\frac{2000}{6000}  \approx 0.333333 \\[10pt]

& P(simple \mid  +) =	\frac{3000}{6000}  = 0.5 \\[10pt]

& P(lot \mid  +) =	\frac{2000}{6000}  \approx 0.333333 \\[10pt]

& P(hate \mid +) = \frac{1}{6000} \approx 0.000167 &(smoothing)\\[10pt]

& P(waste \mid +) = \frac{1}{6000} \approx 0.000167 &(smoothing)\\[10pt]

& P(bore \mid +) = \frac{1}{6000} \approx 0.000167 &(smoothing)\\[10pt]

& P(easy \mid +) = \frac{3000}{6000} = 0.5 \\[10pt]

& P(enjoy \mid +) = \frac{1000}{6000} \approx 0.166667 \\[10pt]

\end{align}

$$

其中,  **hate** , **waste** , **bore** , 在 *sentiment = '+'* 時的 *count* 為 *0* , 為了避免最後相乘的結果變為 *0* , 所以要做 *smoothing* , 也就是說, 將 *count* 改為 *1*  

將以上結果乘起來, 得出以下結果


$$

\begin{align}

&P(+,like,simple,lot,hate',waste',bore',easy',enjoy') \\

&= 0.75 \times  0.333333 \times   0.5\times   0.333333 \times   (1-0.000167) \times   (1-0.000167) \times   (1-0.000167)  

\\& \times 0.5 \times    0.166667 \\

& \approx 0.01716

\end{align}

$$


再來算算看這個句子為負面評論的機率, 得出


$$

P(-,like,simple,lot,hate',waste',bore',easy',enjoy') = 4.048 \times 10^{-7} \approx 0

$$


因為 $$ 0.026 > 0 $$ , 故此句子較有可能為正面評論



## 3.Implementation : Sentiment Analysis


接著來實作一下以上的例子


首先, 開新檔案命名為 *sentiment.py* ,貼上以下內容



```python sentiment.py
_DATA = [ 
{'n':2000., 's':"I really like this course and I learn a lot from it               ", 'p': 1}, 
{'n':800. , 's':"I hate this course and I think it is a waste of time              ", 'p':-1}, 
{'n':200. , 's':"The course is extremely simple and quite a bore                   ", 'p':-1}, 
{'n':3000., 's':"The course is simple, and very easy to follow                     ", 'p': 1}, 
{'n':1000., 's':"I enjoy this course a lot and learning something too              ", 'p': 1}, 
{'n':400. , 's':"I would enjoy myself a lot if i did not have to take this course  ", 'p':-1}, 
{'n':600. , 's':"I didn't enjoy this course                                        ", 'p':-1}, 
]

_KEYWORD = ['like','lot','hate','waste','simple','bore','easy','enjoy']


def smooth(x):
    if x==0.:
        return 1.
    else :
        return x

def pc(c):
    return smooth(sum([d['n'] for d in _DATA if d['p']==c]))/ \
           smooth(sum([d['n'] for d in _DATA]))

def pwc(w,c):
    return smooth(sum([d['n'] for d in _DATA if d['p']==c if w in d['s'] ])) / \ 
           smooth(sum([d['n'] for d in _DATA if d['p']==c]))
        
def psc(s,c):
    return reduce(lambda a,b: a*b, 
           map(lambda w : pwc(w,c) if w in s else 1.-pwc(w,c) , _KEYWORD))*pc(c) 

def sentiment(s):
    if psc(s,1) > psc(s,-1):
       return "positive" 
    elif psc(s,1) < psc(s,-1):
       return "negative" 
    else:
       return "neutral"
       

```


其中, `_DATA` 是我們蒐集的句子 , `_KEYWORD` 是關鍵字 , `pc(c)` , `pwc(w,c)` 和 `psc(s,c)` 是算機率用的 *function* 等下再解釋, `sentiment(s)` 用來算最後的結果, 是正評( *positive* )還是負評( *negative* )


接著到 *python* 的 *interactive mode* 載入這個檔案



```python

>>> import sentiment as st

```


首先, 計算正負評的句子在 `_DATA` 裡面所占的比例, 分別為 $$P(+)$$ 和 $$P(-)$$ , 本程式給正評的 *label*  為 `1` , 附評的 *label* 為 `-1` , 操作方法如下



```python

>>> st.pc(1)
0.75

>>> st.pc(-1)
0.25

```


得出, 正評的句子所占的比例為 `0.75` , 以此類推


接著來算看看, 已知某個句子是正評, 則此文章出現關鍵字 `like` 的機率, 為 $$P(like \mid  +)$$



```python

>>> st.pwc('like',1)
0.3333333333333333

```


再來, 給定一個句子, 如下

$$

\text{I really like this simple course a lot.}

$$

計算這個句子是正評的機率



```python

>>> st.psc('I really like this simple course a lot',1)
0.026028648003351664

```


也可以直接用 `st.sentiment(s)` 判斷這個句子是正評還是負評, 如下



```python

>>> st.sentiment('I really like this simple course a lot')
'positive'

```


再來, 看看負評的例子

$$

\text{I hate this course, it is too boring.}

$$


經由程式計算的結果, 此句為負評, 如下



```python

>>> st.sentiment('I hate this course, it is too boring')
'negative'

```


以上是簡單 *Sentiment Analysis*  實作 


其實, 在現實生活的應用中, *Sentiment Analysis* 的演算法, 還需要考慮到否定詞可以反轉句子的意思, 

例如 *I don't like this course.* 和 *I like this course.* 這兩句的意思是相反的, 但是只看關鍵字, 未必能準確判斷


## 4.Reference


本文內容和範例, 參考至以下這門 *coursera* 的線上課程 

#### Dr. Gautam Shroff.  Web Intelligence and Big Data

https://www.coursera.org/course/bigdata
