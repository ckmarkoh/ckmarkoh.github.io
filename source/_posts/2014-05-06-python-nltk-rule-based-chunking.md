---
layout: post
title: 'NLTK Rule-Based Chunking'
date: 2014-05-06 14:46
comments: true
categories: ['NLTK', 'Natural Language Processing']
---

## Chunking


分析句子的成份, 當句子不符合文法時, 使用 *Context-Free Grammar* 去做 *Parsing* , 可能會得不出結果, 而某些應用, 需要的並不是完整的剖析樹, 而是把句子中某些成份給找出來


*Chunking* 的概念就是, 把句子中的單字分組, 每一組是由一到多個相鄰的單字所組成, 例如, 想要得出句子中有哪些名詞片語, 就可以把相鄰的定冠詞, 名詞修飾語, 以及名詞, 分成一組, 分組所得出的即為名詞片語


舉個例子, 在以下的句子中, 想要找出名詞片語有哪些

$$

\text{The little yellow dog barked at the cat}.

$$

首先, 對這個句子進行 *Part of speech Tagging*

$$

\begin{array}{c}

&\text{The} &\text{little} &\text{yellow} &\text{dog} &\text{barked} &\text{at} &\text{the} &\text{cat} \\

&\text{DT} &\text{JJ} &\text{JJ} &\text{NN} &\text{VBD} &\text{IN} &\text{DT} &\text{NN} 

\end{array}

$$

<!--more-->

*Part of speech Tagging* 完之後可得出這些單字, 就可以根據這些單字的 *Tag* 來決定要怎麼分組, 如果要找名詞片語, 可以把相鄰的 *DT* , *JJ* 和 *NN* 分到名詞片語 *NP* 的組別中 , 如下


$$

\begin{bmatrix}

 &\text{The} &\text{little} &\text{yellow} &\text{dog} \\

 &\text{DT} &\text{JJ} &\text{JJ} &\text{NN} \\

 &\mathbf{NP}

\end{bmatrix}\mspace{10mu}

\begin{array}{c}

 &\text{barked} &\text{at} \\

 &\text{VBD} &\text{IN} \\

 &

\end{array}\mspace{20mu}

\begin{bmatrix}

 &\text{the} &\text{cat} \\

 &\text{DT} &\text{NN} \\

 &\mathbf{NP}

\end{bmatrix}

$$

根據以上結果得出, 名詞片語有 *The little yellow dog* 和 *the cat*


*Chunking* 的演算法, 主要有兩種, 一種是 *Rule-Based*, 另一種是 *Machine-Learning-Based* , 本文的重點放在前者


## Rule-Based Chunking


*Rule-Based Chunking* 是根據事先定好的 *Rule* 來進行 *Chunking*, 例如以上例子, 定義好名詞片語的成份有哪些, 來尋找句子中哪些單字可以組成名詞片語


在 *python nltk* 裡, 可以用 `RegexpParser` 以 *regular expression* 來定義這些 *rule*


首先, 載入模組



```python

>>> from nltk import RegexpParser

```


以及例句



```python

>>> sentence = [("the", "DT"), ("little", "JJ"), ("yellow", "JJ"), ("dog", "NN") \ 
...            ,("barked", "VBD"), ("at", "IN"),  ("the", "DT"), ("cat", "NN")]

```



用 *Regular Expression* 定義好 *NP* 的 *Rule*



```

>>> grammar = "NP: {<DT>?<JJ>*<NN>}"

```


以上的 *rule* 表示, *NP* 是由 *0~1* 個 *DT* , *0* 個以上的 *JJ* 和 一個 *NN* 所組成的 


再來用 `RegexpParser` 進行 *Chunking*, 得出以下結果



```

>>> result = RegexpParser(grammar).parse(sentence)
>>> print result
(S
  (NP the/DT little/JJ yellow/JJ dog/NN)
  barked/VBD
  at/IN
  (NP the/DT cat/NN))

```


也可以把結果畫出來



```python

>>> result.draw()

```

![ch1](/images/pic/pic_00073.png)


如上圖顯示, *Chunking* 的結果, 把屬於 *NP* 的元素都歸到 *NP* 底下了


## Chinking


前文所提到的 *Chunking* 是先定義好, 哪個類別 *有* 哪些元素


但有時候, 想要的取得的類別, 要一一列舉其中的元素, 列舉不完, 如果要用消去法的概念, 來定義哪些元素不包含在這個類別裡面, 就方便多了


而 *Chinking* 的概念是用消去法, 定義哪個類別 **沒有** 哪些元素, 把那些沒有的元素排除, 而剩下來的就被分到該類別

例如可以定義, *NP* 沒有 *VBD* 或 *IN* 這兩者, 如下



```python

>>> grammar2 = r"""
...   NP:
...     {<.*>+}          # Chunk everything
...     }<VBD|IN>+{      # Chink sequences of VBD and IN
...   """

```


其中, `{<.*>+}` 是先把所有東西都包含進去, 再來的 `}<VBD|IN>+{ ` 是用 *Chinking* 把 *VBD* 和 *IN* 這兩者排除, 執行結果如下



```python

>>> result = RegexpParser(grammar2).parse(sentence)
>>> print result
(S
  (NP the/DT little/JJ yellow/JJ dog/NN)
  barked/VBD
  at/IN
  (NP the/DT cat/NN))

```


把結果畫出來



```python

>>> result.draw()

```

![ch1](/images/pic/pic_00073.png)


得出結果和前文的 *Chunking* 相同


# Further Reading


本文是對 *Rule-Based Chunking* 很簡單的介紹

想要更深入了解, 請看

Python nltk Documentation Chunking

http://nltk.googlecode.com/svn/trunk/doc/book/ch07.html#fig-chunk-segmentation


Python nltk HOWTO chunk

http://www.nltk.org/howto/chunk.html
