---
layout: post
title: 'Python nltk -- Dependency Grammar'
date: 2014-05-03 15:00
comments: true
categories: [natural_language_processing, grammar, parsing, dependency_grammar, nltk]
---

## 1. Introduction


在自然語言處理中, *Phase-Structured Grammar* 這類的文法, 是把一個句子, 剖析成一個 **完整的剖析樹** , 它的重點是句子中各個成份的階層關係, 例如 *Context-Free Grammar*


而所謂的 *Dependenct Grammar* , 是著重在 **字和字之間關係** , 而非整個句子中各種成份階層關係, 例如 

$$

\text{I shot an elephant in my pajamas.}

$$

用*Depedenct Grammar* 可以表示成這樣

![d0](/images/pic/pic_00067.png)

如上圖, 把詞語和詞語之間的關係, 用箭頭表示, 箭頭的起點為 *Head*, 終點為 *dependents* , 標示箭頭上的英文字代表 *Head* 和 *Dependent* 之間的 *relation*

<!--more-->

*Head* 的意思是中心語, 就是比較重要的概念, 而 *Dependent* 則是用來修飾這個概念的詞語 , 例如 *an elephant* 這一小片段中, 重要的概念為 *elephant* , 而 *an* 是 *elephant* 的修飾語


通常會在句子中挑一個最重要的中心語, 通常是動詞, 當作 *ROOT* , 如上圖例子的 *ROOT* 為 *shot*, 而 *shot* 的 *Dependent* 有 *I* 和 *elephant* , 它們和 *shot* 之間的 *relation* 分別為 *SBJ* 和 *OBJ* , 以此類推


相較於 *Phase-Structured Grammar* 需要整個句子都文法正確, 才可以得出剖析樹, 而 *Dependency Grammar* 不需要整個句子裡面的每個字都文法正確, 也可以得到剖析樹, 因為只需要注意字和字的關係即可


## 2. DependencyGraph


在 *python nltk* 裡, 可以用 `DependencyGraph` 來載入 *Dependency Grammar*

首先載入模組



```python

>>> from nltk.parse import DependencyGraph

```


接著用以下的 *string* 來表示前例的 *Dependency Grammar*



```python

>>> dep_str= \
... """
... I         NOUN  2    SBJ
... shot      VERB  0    ROOT 
... an        DET   4    DETMOD
... elephant  NOUN  2    OBJ
... in        PREP  4    NMOD
... my        DET   7    DETMOD
... pajamas   NOUN  5    PMOD
... """

>>> 

```


其中, 每行可分成四個部份, 第一部份是句子中的單字, 第二部份是這個單字的 *Part of Speech(POS) Tagging* , 第三和第四部份是它的 *Head* 的 *index* 以及 *relation*, 例如 *I* 這個字, 它的 *POS Tagging* 為 *Noun* , 它的 *Head* 為 *shot*, *Head* 的 *index* 為 *shot* 在句子中的位置, 為 *2*, 而 *relation* 為 *SBJ* , 以此類推, 見下圖

![d1](/images/pic/pic_00068.png)

上圖將每個字標上 *index* , 以方便建構出 *Dependency Graph*


再來用 `DependencyGraph` 讀入這個 `dep_str`



```python

>>> dg = DependencyGraph(dep_str) 

```


可以把 *Dependency Graph* 轉成 *Syntax Tree* 印出來



```python

>>> dg.tree().pprint()
'(shot I (elephant an (in (pajamas my))))'

```


或者畫出來



```python

>>> dg.tree().draw()

```


呈現如下圖

![d2](/images/pic/pic_00070.png)

由上圖可知, *Dependency Grammar* 所形成的 *syntax tree* , 和 *Phase-Structured Grammar* 最大的不同點在於, 每個 *Node* 是一個字, 而不是 *Non-Terminal* , *Node* 和 *Node* 之間的 *edge* 表示 *dependent relation* 而非 *production*. 



## 3. Dependency Parsing


接著來談到如何把一個句子, 利用 *Dependency Grammar* 轉成 *Syntax Tree* 和 *Dependency Graph*


先載入模組



```python

>>> from nltk import parse_dependency_grammar

```


還有尚未剖析的 *Raw Sentence*



```python

>>> sent = 'I shot an elephant in my pajamas'.split()

```


接著是 *Grammar* 的部份, 箭號左邊為 *head* , 右邊為 *dependent*



```python

>>> dep_grammar = parse_dependency_grammar(
... """
... 'shot' -> 'I' | 'elephant' | 'in'
... 'elephant' -> 'an' | 'in'
... 'in' -> 'pajamas'
... 'pajamas' -> 'my'
... """
... )

```


在 `nltk` 裡面, 提供兩種方式, 分別為 *Projective Dependency* 和 *Non-Projective Dependency*


分別可以把 *Raw Sentence* 轉為 *Syntax Tree* 和 *Dependency Graph*


### 3.1 Projective Dependency Parsing


`ProjectiveDependencyParser` 可將 *Raw Sentence* 轉為 *syntax tree*


載入以下模組



```python

>>> from nltk import ProjectiveDependencyParser

```


接著用 `ProjectiveDependencyParser` 載入文法來剖析句子, 如下



```python

>>> trees = ProjectiveDependencyParser(dep_grammar).parse(sent)

```


因為本例的文法為 *ambiguous grammar* , 也就是說, 可能的 *syntax tree* 超過一種以上

程式會求出所有可能的 *syntax tree* , 如本例, 有兩個可能的 *syntax tree* 



```

>>> trees[0].pprint()
'(shot I (elephant an (in (pajamas my))))'

>>> trees[1].pprint()
'(shot I (elephant an) (in (pajamas my)))'

```


用以下方法畫出來



```python

>>> trees[0].draw()
>>> trees[1].draw()

```

![d2](/images/pic/pic_00070.png)

![d3](/images/pic/pic_00071.png)


### 3.2 Non-Projective Dependency Parsing


`NonprojectiveDependencyParser` 可將 *Raw Sentence* 轉為 *dependency graph*

載入模組



```python

>>> from nltk import NonprojectiveDependencyParser

```


接著用 `NonprojectiveDependencyParser` 載入文法來剖析句子, 如下




```python

>>> dep_graphs = NonprojectiveDependencyParser(dep_grammar).parse(sent)

```


本例也會得出兩個可能的結果, 可以用 `print` 印出 *dependency graph*



```python

>>> print dep_graphs[0]
[{'address': 0, 'deps': 2, 'rel': 'TOP', 'tag': 'TOP', 'word': None},
 {'address': 1, 'deps': [], 'word': 'I'},
 {'address': 2, 'deps': [1, 4], 'word': 'shot'},
 {'address': 3, 'deps': [], 'word': 'an'},
 {'address': 4, 'deps': [3, 5], 'word': 'elephant'},
 {'address': 5, 'deps': [7], 'word': 'in'},
 {'address': 6, 'deps': [], 'word': 'my'},
 {'address': 7, 'deps': [6], 'word': 'pajamas'}]

>>> print dep_graphs[1]
[{'address': 0, 'deps': 2, 'rel': 'TOP', 'tag': 'TOP', 'word': None},
 {'address': 1, 'deps': [], 'word': 'I'},
 {'address': 2, 'deps': [1, 4, 5], 'word': 'shot'},
 {'address': 3, 'deps': [], 'word': 'an'},
 {'address': 4, 'deps': [3], 'word': 'elephant'},
 {'address': 5, 'deps': [7], 'word': 'in'},
 {'address': 6, 'deps': [], 'word': 'my'},
 {'address': 7, 'deps': [6], 'word': 'pajamas'}]

```


其中 `address` 表示 `word` 的 *id* , `deps` 為這個 `word` 的 *dependent*


*註：事實上,  Non-Projective 和 Projective 最主要的差異在於, Non-Projective 可以處理字詞順序可任意顛倒的語言, 在處理這種語言時, 產生出的 Syntax Tree 由於字詞顛倒, 會有交叉, 故通常用 Dependency Graph 來表示結果, 本文不探討這種現象, 若你想了解, 請看以下的 Further Reading* 


## Furtuer Reading


想要了解更多關於 *Dependency Grammar* 可以看以下兩篇 *python nltk* 的 *Documentation*


Python nltk HOWTO dependency

http://www.nltk.org/howto/dependency.html


Python nltk book ch08

http://nltk.googlecode.com/svn/trunk/doc/book/ch08.html