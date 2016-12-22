---
layout: post
title: 'Minimum Edit Distance'
date: 2014-10-11 14:28
comments: true
categories: ['Natural Language Processing','Text Mining']
---

## Minimum Edit Distance


*Minimum Edit Distance* （最小編輯距離），是用來計算兩個字串的相似程度。

可應用於拼字校正、或是計算兩個DNA序列的相似程度。


例如，假設有三個DNA序列：


    1. AGCCT

    2. AACCT

    3. ATCT


直覺上，會認為，DNA序列 *2.* 和 *1.* 的相似程度比 *3.* 和 *1.* 的相似程度還要高。

但為何是 *2.* 和 *1.* 較相似？相似程度又是多少？

若要把兩序列的相似程度量化成數值，就要計算「最小編輯距離」，最小編輯距離越小，就表示兩序列越相似。


## Defining of Minimum Edit Distance


我們採用 *Levenshtein* 的最小編輯距離的定義，這是一種最簡單的定義方式。依此定義，若一個字串，編輯成另外一個字串，可以進行下面三種動作。


<!--more-->


$$

\begin{array}{c  c }

		operation   &  cost \\ \hline

	a.插入 (insertion) & 1 \\

	b.刪除 (deletion)   & 1  \\

	c.替換 (substitution) & 2  \\

\end{array}

$$

其中，定義 *a.* 和 *b.* 的 *cost* 為 $$1$$ ， *c.* 的 *cost* 為 $$2$$ （因為「替換」相當於「刪除」後再「插入」）。


例如， *1.* 和 *2.* 的最小編輯距離，可以這樣計算，如下：

$$

\begin{array}{ c c c c c}

    \ A & G & C & C & T \\

    \ \mid & \mid & \mid & \mid & \mid \\

    \ A & A & C & C & T \\

		\   &  s &  &  &  &\\

\end{array}

$$

上圖中，將兩字串對齊，發現要把 *1.* 編輯成 *2.* ，需要對第二個字 *G* 進行替換（ *s* ），把 *G* 替換成 *A* ，而替換的 *cost* 為 $$2$$，故 *1.* 和 *2.* 的最小編輯距離為 $$2$$。

 

再來，把 *1.* 編輯成 *3.* 的最小編輯距離，如下：

$$

\begin{array}{ c c c c c}

    \ A & G & C & C & T \\

    \ \mid & \mid & \mid & \mid & \mid \\

    \ A & \Box & T & C & T \\

		\ &  d & s &  &  &\\

\end{array}

$$

上圖中， $$\Box$$ 表示該處沒字元， 而要把 *1.* 編輯成 *3.* ，需要把第二個字 *G* 刪除（ *d* ），並替換（ *s* ）第三個字 *C* ，這兩個動作的 *cost* 分別為 $$1$$ 和 $$2$$ ，故最小編輯距離為 $$1+2 =3$$


## Computing Minimum Edit Distance


接著來看，要用什麼演算法，來計算最小編輯距離？可以用 *Dynamic Programming* 的方式來計算。


所謂的 *Dynamic Programming* 簡而言之，就是把一個問題，一層一層分解下去，分成許多子問題，再算出這些子問題，用這些子問題的解答，求出原本問題的答案。


首先，定義字串長度為 $$m,n$$ 的兩字串，最小編輯距離為 $$D(m,n)$$


在分解過程中，需要計算長度為 $$i,j$$的兩個子字串，最小編輯距離為 $$D(i,j)$$ ，其中 $$0<i<m, 0<j<n$$ 。


接著，定義起始狀態，就是把兩個給定的字串，各自編輯成空字串的距離，分別為 $$D(i,0)$$ 和 $$D(0,j)$$。

由於，把字串長度為 $$i$$ 的字串，編輯成空字串長度為 $$i$$ ，即 $$D(i,0)=i$$ , 同理， $$D(0,j)=j$$


演算法如下：從起始狀態開始，由 $$i=1\cdots m$$ 及 $$j=1\cdots n$$ ，依序去計算長度為 $$i,j$$ 字串的最小編輯距離。


$$

\begin{align}

&\text{for each } i = 1\cdots m \\

&\mspace{20mu}\text{for each } j = 1\cdots n \\

&\mspace{40mu} D(i,j) = min 

	\left\{

	\begin{array}{l}

	D(i-1,j) + 1 \\

	D(i,j-1) + 1 \\

  D(i-1,j-1) + 	\left\{\begin{array}{l}

									 2 & \text{if} X(i) \neq Y(j) \\

 									 0 & \text{if} X(i) = Y(j) \\

                	\end{array}\right.  

	\end{array}

\right.  

\end{align}

$$

其中， $$X(i)$$ 為第一個字串中第 $$i$$ 個字元，$$Y(j)$$ 為第二個字串中第 $$j$$ 個字元。


用以上演算法，舉個例子來計算，若兩字串分別為：

1. AGCCT

2. ATCT


輸入字串長度分別為 $$5$$ 和 $$4$$，故 $$m=5,n=4$$。

演算法開始的第一步，先建立以下表格，右下方空格部份即為 $$D(i,j)$$ 的值。 


$$

	\begin{array}{|c|c|c|c|c|c|c|}

  \hline

	          \setminus & j  & 0    & 1 & 2 & 3 & 4  \\ \hline

  					i	& \setminus  & \Box & A & T & C & T  \\ \hline 

            0   & \Box     &      &   &   &   &    \\  \hline

            1   & A        &      &   &   &   &    \\  \hline

            2   & G        &      &   &   &   &    \\ \hline

            3   & C        &      &   &   &   &    \\ \hline

            4   & C        &      &   &   &   &    \\ \hline

            5   & T        &      &   &   &   &    \\ \hline

	\end{array}

$$


接著，填入初始狀態的值，即 $$D(i,0)=i, D(0,j)=j$$


$$

	\begin{array}{|c|c|c|c|c|c|c|}

  \hline

              \setminus & j  & 0    & 1 & 2 & 3 & 4  \\ \hline

            i   & \setminus  & \Box & A & T & C & T  \\ \hline

            0   & \Box     & \color{blue}{0}   & \color{blue}{1} & \color{blue}{2} & \color{blue}{3} & \color{blue}{4}  \\ \hline

            1   & A        & \color{blue}{1}   &   &   &   &    \\ \hline

            2   & G        & \color{blue}{2}  &   &   &   &    \\ \hline

            3   & C        & \color{blue}{3}  &   &   &   &    \\ \hline

            4   & C        & \color{blue}{4}  &   &   &   &    \\ \hline

            5   & T        & \color{blue}{5}  &   &   &   &    \\ \hline

	\end{array}

$$


再來，計算 $$D(1,1)$$ 的值，用演算法的公式，如下：


$$

\begin{align}

& D(1,1) \\

&= min 

	\left\{

	\begin{array}{l}

	D(0,1) + 1 \\

	D(1,0) + 1 \\

  D(0,0) + 	\left\{\begin{array}{l}

									 2 & \text{if} X(1) \neq Y(1) \\

 									 0 & \text{if} X(1) = Y(1) \\

                	\end{array}\right.  

	\end{array}

\right. \\

&  = min 

	\left\{

	\begin{array}{l}

	1 + 1 \\

	1 + 1 \\

  0 + 0  

	\end{array}

\right. \\

& = 0 

\end{align}

$$


其中，可從表格中得知 $$ D(0,1) = 1 , D(1,0) = 1 , D(0,0) = 0 $$，又因為 $$X(1) = Y(1) = \text{A}$$ ，故 $$ D(1,1)$$ 的最小編輯距離為 $$ 0+0=0 $$ 。將算出來的 $$D(1,1)$$ 結果，填入表格，如下：


$$

	\begin{array}{|c|c|c|c|c|c|c|}

  \hline

              \setminus & j  & 0    & 1 & 2 & 3 & 4  \\ \hline

            i   & \setminus  & \Box & A & T & C & T  \\ \hline

            0   & \Box     & \color{gray}{0}   & \color{gray}{1} & \color{gray}{2} & \color{gray}{3} & \color{gray}{4}  \\ \hline 

            1   & A        & \color{gray}{1}  &  \color{blue}{0} &   &   &    \\ \hline

            2   & G        & \color{gray}{2}  &   &   &   &    \\ \hline

            3   & C        & \color{gray}{3}  &   &   &   &    \\ \hline

            4   & C        & \color{gray}{4}  &   &   &   &    \\ \hline

            5   & T        & \color{gray}{5}  &   &   &   &    \\ \hline

	\end{array}

$$

用此公式，依序算出表格中其他空格的值。


$$

	\begin{array}{|c|c|c|c|c|c|c|}

  \hline

              \setminus & j  & 0    & 1 & 2 & 3 & 4  \\ \hline

                    i   & \setminus  & \Box & A & T & C & T  \\ \hline

            0   & \Box     & \color{gray}{0}  & \color{gray}{1} & \color{gray}{2} & \color{gray}{3} & \color{gray}{4} \\ \hline 

            1   & A        & \color{gray}{1}  & \color{gray}{0} & \color{gray}{1} & \color{gray}{2} & \color{gray}{3}  \\ \hline 

            2   & G        & \color{gray}{2}  & \color{gray}{1} & \color{gray}{2} & \color{gray}{3} & \color{gray}{4}  \\ \hline 

            3   & C        & \color{gray}{3}  & \color{gray}{2} & \color{gray}{3} & \color{gray}{2} & \color{gray}{3}  \\ \hline 

            4   & C        & \color{gray}{4}  & \color{gray}{3} & \color{gray}{4} & \color{gray}{3} & \color{gray}{4}  \\ \hline 

            5   & T        & \color{gray}{5}  & \color{gray}{4} & \color{gray}{5} & \color{gray}{4} & \color{blue}{3}  \\ \hline

	\end{array}

$$


全部算完以後，表格中最右下方的數字，即為 AGCCT 和 ATCT 兩序列的最小編輯距離，其值為3。


## Implementation


再來是實作的部份，建立一個python程式檔 *minEditDist.py* ，並將以下程式複製貼上。



```python minEditDist.py
def minEditDist(sm,sn):
  m,n = len(sm),len(sn)
  D = map(lambda y: map(lambda x,y : y if x==0 else x if y==0 else 0,
    range(n+1),[y]*(n+1)), range(m+1))
  for i in range(1,m+1):
    for j in range(1,n+1):
      D[i][j] = min( D[i-1][j]+1, D[i][j-1]+1, 
        D[i-1][j-1] + apply(lambda: 0 if sm[i-1] == sn[j-1] else 2)) 
  for i in range(0,m+1):
    print D[i] 
  return D[m][n] 

```


其中， `sm` 和 `sn` 分別為輸入的字串，而 `D[i][j]` 為  最小編輯距離 $$D(i,j)$$。

到 *python interactive mode* 載入模組



```python

>>> from minEditDist import minEditDist

```


接著，用此程式計算 *AGCCT* 和 *ATCT* 的最小編輯距離，方法如下：



```python

>>> minEditDist("AGCCT","ATCT")
[0, 1, 2, 3, 4]
[1, 0, 1, 2, 3]
[2, 1, 2, 3, 4]
[3, 2, 3, 2, 3]
[4, 3, 4, 3, 4]
[5, 4, 3, 4, 3]
3

```


程式印出 $$D(i,j)$$ 表格中的數值，並回傳最後結果，結果為 $$3$$ 。


## Reference


本文參考至coursera線上課程

#### Dan Jurafsky & Christopher Manning. Natural Language Processing

https://www.coursera.org/course/nlp
