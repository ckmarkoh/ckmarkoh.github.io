---
layout: post
title: 'NLTK Tree'
date: 2014-02-28 16:36
comments: true
categories: ['Natural Language Processing']
---


今天我們來看看nltk.Tree要怎麼用


先載入模組


```python

>>> from nltk import Tree

```



## 1.build syntax tree


舉個例子

Gary plays baseball

此句子的剖析樹(syntax tree)是這樣：

<img src="http://lh3.googleusercontent.com/-BoNaoH3Va-g/UyF3tY8MRiI/AAAAAAAAAXM/1-YLB7zww3k/w299-h238-no/nltk1.png" alt="nltk1.png">


用nltk.Tree來建構tree：


```python

>>> tree=Tree('S',[Tree('NP',['Gary']),
...           Tree('VP',[Tree('VT',['plays']),
...                     Tree('NP',['baseball'])])])

>>> tree.draw()

```


即可將剖析樹畫出來

若沒安裝 python-tk,你也可以這樣把tree印出



```python

>>> tree.pprint()
'(S (NP Gary) (VP (VT plays) (NP baseball)))'

```

<!--more-->

## 2. Subtrees ,Nodes and Leaves


再來可以對tree進行操作

例如：取得tree的subtree, node 和leaf



```python

>>> tree[1]
Tree('VP', [Tree('VT', ['plays']), Tree('NP', ['baseball'])])

>>> tree[1].node
'VP'

>>> tree[1,1]
Tree('NP', ['baseball'])

>>> tree[1,1].node
'NP'

>>> tree[1,1,0]
'baseball'

>>> tree.leaves()
['Gary', 'plays', 'baseball']

```


## 3. Grammar , Chomsky normal form


我們也可以看看這個tree是由哪些grammar產生的

可以用`productions()`得出grammar



```python

>>> tree.productions()
[S -> NP VP, NP -> 'Gary', VP -> VT NP, VT -> 'plays', NP -> 'baseball']

```


再來,我們可以把grammar轉成chomsky normal form(CNF)

（若不清楚CNF是什麼,請查閱"計算理論"的教科書)

首先,看看一個不符合CNF的例子:



```python

>>> tree2= Tree('S', [ Tree('NP', ['Gary']),
...                     Tree('VT', ['play']), 
...                     Tree('NP', ['baeball'])])

>>> tree2.productions()
[S -> NP VT NP, NP -> 'Gary', VT -> 'play', NP -> 'baeball']

>>> tree2.draw()

```

<img src="http://lh3.googleusercontent.com/-RO644KujeOY/UyF3tRi2mfI/AAAAAAAAAXA/i1iPNbISnaA/w285-h189-no/nltk2.png" alt="nltk2.png">


`S -> NP VT NP`不符合chomsky normal form

可以用`chomsky_normal_form()`



```python

>>> tree2.chomsky_normal_form()
>>> tree2.productions()
[S -> NP S|<VT-NP>, NP -> 'Gary', S|<VT-NP> -> VT NP, VT -> 'play', NP -> 'baeball']

>>> tree2.draw()

```

<img src="http://lh5.googleusercontent.com/-aW-KwndtZOg/UyF3tQfi-jI/AAAAAAAAAXE/j4fpXq2-9Ro/w273-h234-no/nltk3.png" alt="nltk3.png">

轉換後產生了一個新的node,`S|<VT-NP>`,這樣子就符合CNF了


## 4.Parse tree from string


如果我們現在有一個string如下：



```python

>>> s=r"(S (NP Gary) (VP (VT plays) (NP baseball)))"

```


要從這個string建立出tree,可用以下方法：



```python

>>> tree3=Tree.parse(s)
>>> tree3.pprint()
'(S (NP Gary) (VP (VT plays) (NP baseball)))'

```


## 結語：


想要看更多參考資料,請到：

tutorial:

https://nltk.googlecode.com/svn/trunk/doc/howto/tree.html

http://www.mit.edu/~6.863/spring2011/labs/nltk-tree-pages.pdf

api documentation:

http://nltk.googlecode.com/svn/trunk/doc/api/nltk.tree.Tree-class.html#productions

source code:

http://www.nltk.org/_modules/nltk/tree.html
