---
layout: post
title: '資訊檢索 -- Boolean Retrieval'
date: 2015-02-09 00:27
comments: true
categories: 
---

## Information Retrieval


所謂的資訊檢索（ *Information Retrieval* )，就是從大量非結構的資料，例如網頁，根據某些關鍵字，找出具有此關鍵字的文件。例如，搜尋引擎，即是一種資訊檢索的應用。


資訊檢索的演算法，其實跟我們要在某本書中，找尋某個字的方法差不多。

例如我們想在 *Introduction to Information Retrieval* 這本教科書中，找到 *informational queries* 這個詞在哪一頁，如果從第一頁開始，一個字一個字慢慢找，要比對成千上萬個字才能找到。


但如果我們翻到書本後面的 *Index* （如下） ，即可很快找到 *informational queries* 這個字是在第 *432* 頁。


```
...
information gain, 285 
information need, 5, 152 
information retrieval, 1 
informational queries, 432 
inner product, 121 
...

```


所以，資訊檢索，最核心的概念就是建立 *Index* ，這樣就可以快速找到哪些文件中具有某個關鍵字。

<!--more-->


## Boolean Retrieval


最基本的資訊檢索方法，就是 *Boolean Retrieval* 。它用 *boolean logic* 的概念，來建立 *index* 和執行 *query* 。


舉個例子，假設有三篇文劍，內容分別如下：


|文件 | 內容 | 
|----|-----------------------------------------------------------|
| D1 | The way to avoid linearly scanning is to index the documents in advance. |
| D2 | The model views each document as just a set of words.  |
| D3 | We will discuss and model these size assumption. | 


根據這些文字，我們可以建立 *Term-Document Matrix* ，記錄哪個字出現在哪篇文章，如下：


|Term | D1 | D2 | D3|
|----|----|----|----|
| way | 1  | 0 | 0 |
| avoid | 1 | 0 | 0 |
| linear | 1 | 0 | 0 |
| scan | 1 | 0 | 0 |
| index | 1 | 0 | 0 |
| document | 1 | 1 |  0 |
| advance | 1 |  0 | 0 |
| model | 0 |  1 | 1 | 
| view | 0 | 1 |  0 |
| set | 0 | 1 |  0 |
| word | 0 | 1 |  0 |
| discuss |0 | 0| 1 |
| size | 0 | 0 | 1 |
| assumption | 0 | 0 | 1 |


此表格用 *boolean* 值來表示某個字出現在哪幾篇文章，例如， *way* 出現在文件 *D1* ，而沒出現在 *D2* 和 *D3* ，所以它的值為 *1 0 0* 。


為了避免此表格過大，建立過多無意義的 *index* ，因此運用了以下兩個原則：

    1. 忽略 *the* , *a* 這些高頻字（ 這種字稱為 *stop word* ）

    2. 將有形態變化的字轉為原型，例如把 *views* 轉為 *view* （ 此過程稱為 *stemming* ） 


建立了 *Term-Document Matrix* 之後，就可以用關鍵字來查詢


**1.查詢 *way* 這個字，可得出 *way* 的值為：**


$$

way = [1,0,0] 

$$


所以 *way* 出現在 *D1* 。


**2.查詢沒有 *way* 這個字的，可用以下的 *boolean* 運算得出結果：**


$$

\neg way = \neg ([1,0,0]) = [0 ,1, 1]

$$

所以沒有 *way* 的文件為 *D2* 、 *D3* 。


**3.查詢包含 *document* 和 *model* 這兩個字都有文件：**


$$

document \wedge model  = [1,1,0] \wedge [0,1,1] = [0 ,1, 0]

$$


得出結果為 *D2* 。


**4.查詢有 *avoid* 或 *view* 其中一個字的文件：**


$$

avoid \vee view  = [1,0,0] \vee [0,1,0] = [1 ,1, 0]

$$


得出結果為 *D1* 和 *D2* 。


## Reference


本文參考教科書 *Introduction to Information Retrieval*

http://nlp.stanford.edu/IR-book/