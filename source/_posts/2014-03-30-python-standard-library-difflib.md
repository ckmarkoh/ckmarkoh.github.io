---
layout: post
title: 'Python Standard Library -- difflib'
date: 2014-03-30 04:24
comments: true
categories: [python_standard_library]
---


## 1.Intoduction


今天來看看 python 的 *difflib* ,

*difflib* 常常用於字串處理,

例如可以用來比較兩個 *string* , 或兩個 *list* 有哪些不一樣

<!--more-->

首先,載入模組：



```python

>>> from difflib import *

```



## 2.contet_diff


要比較兩個檔案裡面有哪幾行不一樣, 可以用 `contet_diff`

假設已經先把檔案讀進來了, 存成 *s1* 和 *s2* 兩個 *list*

則可以用這個方式做比較



```python

>>> s1 = ['aa','bb','cc','dd']
>>> s2 = ['aaa','bb','dd','ee']
>>> g1 = context_diff(s1, s2, fromfile='s1', tofile='s2')
>>> print g1
<generator object context_diff at 0xb72803c4>

```

其中 `fromfile` 和 `tofile` 

只是標示這兩個 *list* 的名稱而已,

和檔案讀取沒什麼關係


執行完以後會產生一個 *generator* 

而 *generator* 是一種 *Design Pattern* 

在此不提相關理論

其實只要把它當成 *iterable* 用 *for*  迴圈印出來就可以了

如下：



```python

>>> for line in g1:
...     print line
... 
*** s1

--- s2

***************

*** 1,4 ****

! aa
  bb
- cc
  dd
--- 1,4 ----

! aaa
  bb
  dd
+ ee

```


印出結果後, 每一行前面會有一些符號, 例如 `-` , `+` ,所代表的意思如下：


|符號|意思| 
|---|---|
|'-'|這一行只在 *s1*|   
|'+'|這一行只在 *s2*|   
|'&nbsp;&nbsp;'|這一行, *s1* 和 *s2* 都有,且都一樣|
|'!'|這一行, *s1* 和 *s2* 都有,但不太一樣|
|'?'|這一行, *s1* 和 *s2* 都沒有|


例如, `aa` 和 `aaa` 分別在 *s1* 和 *s2* , 但這兩行不太一樣, 所以標示出 '!'

而 `cc`  只存在於 *s1* , 所以標示出 '-' 


## 3.unified_diff


和 `contet_diff` 一樣, 

`unified_diff` 也可以用來比較兩個 *list* 的差異

只是結果顯示的方式不一樣, 

不會把兩個 *list* 的內容分開顯示,

而是會合併起來一起顯示



```python

>>> g2 = unified_diff(s1, s2, fromfile='s1', tofile='s2')
>>> for line in g2:
...     print line
... 
--- s1

+++ s2

@@ -1,4 +1,4 @@

-aa
+aaa
 bb
-cc
 dd
+ee

```


## 4.ndiff


`ndiff` 也是用來比較兩個 *list* 有哪裡不同,

和 `contet_diff` 以及 `unified_diff` 最大的差別在於

`ndiff` 會標示出某個 *element* 哪邊不一樣,

要怎麼修改,才會使兩個 *list* 看起來一樣


比較以下兩個 *list* 的異同



```python

>>> s3 = ['aaa','bbb','ccc','ddfd','eeee']
>>> s4 = ['aa','bbbb','ccc','dded','ffff']
>>> for line in g3:
...     print line
... 

>>> g3 = ndiff(s3,s4)

```


用 *for* 迴圈印出 *generator* 的結果：



```

>>> for line in g3:
...     print line
... 
- aaa
?   -

+ aa
- bbb
+ bbbb
?    +

  ccc
- ddfd
?   ^

+ dded
?   ^

- eeee
+ ffff

>>> 

```


其中, `?   -` , `?    +` 和 `?   ^` ,

標示出了 *s3* 的某個 *element* ,和 *s4* 的某個 *element* ,哪邊不一樣,


## 5.get_close_matches


`get_close_matches` 是用來尋找某個 *input string* 和哪個 *list* 裡面的 *element* 最相近

這可以用來做拼字錯誤的修正

例如：



```python

>>> get_close_matches('appel', ['apple', 'bird', 'chair'])
['apple']

>>> get_close_matches('appel', ['apples', 'apple', 'bird', 'chair'])
['apple', 'apples']

```


結果會顯示出一個或多個 *element*, 並依據相似程度排序



## 6.SequenceMatcher


接著來看看 *SequenceMatcher* 這個 *Class*




#### 6.1.find_longest_match


`find_longest_match`  就是 *Longest Common Subsequence(LCS) *

就是在演算法課程中提到的DNA定序的演算法

假設有 `a1` , `b1` 兩個序列, 則方法如下： 



```python

>>> a1 = "ATGCATAA"
>>> b1 = "CCTGCATC"
>>> match1 = SequenceMatcher(None, a1, b1).find_longest_match(0, len(a1), 0, len(b1))
>>> print match1
Match(a=1, b=2, size=5)

```


`Match` 中的值, 分別是 *LCS* `a1` , `b1` 兩序列的起點, 以及 *LCS* 的長度

可用以下方法把 *LCS* 從 `a1` , `b1` 中取出



```python

>>> a1[match1[0]:match1[0]+match1[2]]
'TGCAT'

>>> b1[match1[1]:match1[1]+match1[2]]
'TGCAT'

```



####  6.2.get_matching_blocks


`get_matching_blocks` 可以取出兩個 sequence 中數個 match 的部份,

如下：



```python

>>> a2 = "ATGAACTCTTAAGA"
>>> b2 = "AATGCCAGACTTAC"
>>> match2 = SequenceMatcher(None, a2, b2).get_matching_blocks()
>>> match2
[Match(a=0, b=1, size=3), Match(a=3, b=6, size=1), Match(a=4, b=8, size=1), Match(a=7, b=9, size=4), Match(a=14, b=14, size=0)]

```


結果會回傳多個 match 的 subsequence

可用以下方法印出來看看



```python

>>> for m in match2:
...     print a2[m[0]:m[0]+m[2]],b2[m[1]:m[1]+m[2]]
... 
ATG ATG
A A
A A
CTTA CTTA

```


將match的部份標出來：


<p>

<SPAN STYLE="text-decoration: none"><FONT FACE="Arial, serif">a2=&quot;</FONT><FONT style="background:#ff00ff !important;">ATG</FONT><FONT style="background:#00ffff !important;">A</FONT><FONT style="background:#00ff00 !important;">A</FONT>CT<FONT style="background:#ffff00 !important;">CTTA</FONT>AGA&quot;</SPAN>

</p>

<p>

<SPAN STYLE="text-decoration: none"><FONT FACE="Arial, serif">b2=&quot;A</FONT><FONT style="background:#ff00ff !important;">ATG</FONT>CC<FONT style="background:#00ffff !important;">A</FONT>G<FONT style="background:#00ff00 !important;">A</FONT><FONT style="background:#ffff00 !important;">CTTA</FONT>C&quot;</SPAN>

</p>


如果把 `a2` 和 `b2` 對調,

會發現得出的結果不太一樣



```python

>>> match3 = SequenceMatcher(None, b2, a2).get_matching_blocks()
>>> for m in match3:
...     print b2[m[0]:m[0]+m[2]],a2[m[1]:m[1]+m[2]]
... 
ATG ATG
C C
CTTA CTTA

```


將match的部份標出來：


<p>

<SPAN STYLE="text-decoration: none"><FONT FACE="Arial, serif">b2=&quot;A</FONT><FONT style="background:#ff00ff !important;">ATG</FONT><FONT style="background:#00ffff !important;">C</FONT>CAGA<FONT style="background:#00ff00 !important;">CTTA</FONT>C&quot;</SPAN>

</p>


<p>

<SPAN STYLE="text-decoration: none"><FONT FACE="Arial, serif">a2=&quot;</FONT><FONT style="background:#ff00ff !important;">ATG</FONT>AA<FONT style="background:#00ffff !important;">C</FONT>T<FONT style="background:#00ff00 !important;">CTTA</FONT>AGA&quot;</SPAN>

</p>


至於為什麼會這樣呢？

可能是因為演算法是先從第一個sequence逐次比對


#### 6.3.get_opcodes


跟 `get_matching_blocks` 一樣, 

`get_opcodes` 也可以取出兩個 sequence 中數個 match 的部份,

所得出的 match部份也一樣, 

唯一不一樣之處,在於 `get_opcodes` 會顯示出, 

要怎麼把 `a` 修正, 才會和 `b` 一樣



```python

>>> opc1=SequenceMatcher(None, a2, b2).get_opcodes()
>>> for tag, i1, i2, j1, j2 in  opc1:
...     print "%7s a[%d:%d] (%s) b[%d:%d] (%s)" %(tag, i1, i2, a2[i1:i2], j1, j2, b2[j1:j2])
... 
 insert a[0:0] () b[0:1] (A)
  equal a[0:3] (ATG) b[1:4] (ATG)
 insert a[3:3] () b[4:6] (CC)
  equal a[3:4] (A) b[6:7] (A)
 insert a[4:4] () b[7:8] (G)
  equal a[4:5] (A) b[8:9] (A)
 delete a[5:7] (CT) b[9:9] ()
  equal a[7:11] (CTTA) b[9:13] (CTTA)
replace a[11:14] (AGA) b[13:14] (C)

```

其中 `equal` 是指兩個sequence一樣的部份

把 `equal` 的部份標示出來, 

跟剛剛的 `get_matching_blocks` 結果一樣


另外, `insert` , `delete` 和 `replace` 是修正方法

例如 `insert a[0:0] () b[0:1] (A)` 就是把 `a2[0:0]` 的部份插入 `(A)`

做完這一步以後會變成這樣,

a2 = "AATG..."

b2 = "AATG..."


以下為 `equal` 的部份：


<p>

<SPAN STYLE="text-decoration: none"><FONT FACE="Arial, serif">a2=&quot;</FONT><FONT style="background:#ff00ff !important;">ATG</FONT><FONT style="background:#00ffff !important;">A</FONT><FONT style="background:#00ff00 !important;">A</FONT>CT<FONT style="background:#ffff00 !important;">CTTA</FONT>AGA&quot;</SPAN>

</p>

<p>

<SPAN STYLE="text-decoration: none"><FONT FACE="Arial, serif">b2=&quot;A</FONT><FONT style="background:#ff00ff !important;">ATG</FONT>CC<FONT style="background:#00ffff !important;">A</FONT>G<FONT style="background:#00ff00 !important;">A</FONT><FONT style="background:#ffff00 !important;">CTTA</FONT>C&quot;</SPAN>

</p>



再來, 我們也可以把 `a2` 和 `b2` 順序對調, 

也可得出不太一樣的結果,如下：



```python

>>> opc2=SequenceMatcher(None, b2, a2).get_opcodes()
>>> for tag, i1, i2, j1, j2 in  opc2:
...     print "%7s b[%d:%d] (%s) a[%d:%d] (%s)" %(tag, i1, i2, b2[i1:i2], j1, j2, a2[j1:j2])
... 
 delete b[0:1] (A) a[0:0] ()
  equal b[1:4] (ATG) a[0:3] (ATG)
 insert b[4:4] () a[3:5] (AA)
  equal b[4:5] (C) a[5:6] (C)
replace b[5:9] (CAGA) a[6:7] (T)
  equal b[9:13] (CTTA) a[7:11] (CTTA)
replace b[13:14] (C) a[11:14] (AGA)

```


標示出 `equal` 的部份：


<p>

<SPAN STYLE="text-decoration: none"><FONT FACE="Arial, serif">b2=&quot;A</FONT><FONT style="background:#ff00ff !important;">ATG</FONT><FONT style="background:#00ffff !important;">C</FONT>CAGA<FONT style="background:#00ff00 !important;">CTTA</FONT>C&quot;</SPAN>

</p>


<p>

<SPAN STYLE="text-decoration: none"><FONT FACE="Arial, serif">a2=&quot;</FONT><FONT style="background:#ff00ff !important;">ATG</FONT>AA<FONT style="background:#00ffff !important;">C</FONT>T<FONT style="background:#00ff00 !important;">CTTA</FONT>AGA&quot;</SPAN>

</p>




#### 6.4.match lists of strings


其實, `SequenceMatcher` 也可以用來比對兩個 *list* 中的 *element* 一不一樣



```python

>>> l1=["ab","cd","x","cd"]
>>> l2=["ab","cd","cd"]
>>> s4 = SequenceMatcher(None, l1,l2)

```


##### 6.4.1.find_longest_match



```python

>>> match4=s4.find_longest_match(0, len(l1), 0, len(l2))
>>> l1[match4[0]:match4[0]+match4[2]]
['ab', 'cd']

>>> l2[match4[1]:match4[1]+match4[2]]
['ab', 'cd']

```


##### 6.4.2.find_longest_match



```python

>>> match5=s4.get_matching_blocks()
>>> for m in match5:
...     print l1[m[0]:m[0]+m[2]],l2[m[1]:m[1]+m[2]]
... 
['ab', 'cd'] ['ab', 'cd']
['cd'] ['cd']
[] []

>>> 

```


##### 6.4.3.get_opcodes



```python

>>> opc3=s4.get_opcodes()
>>> for tag, i1, i2, j1, j2 in  opc3:
...     print "%7s l1[%d:%d] (%s) l2[%d:%d] (%s)" %(tag, i1, i2, l1[i1:i2], j1, j2, l2[j1:j2])
... 
  equal l1[0:2] (['ab', 'cd']) l2[0:2] (['ab', 'cd'])
 delete l1[2:3] (['x']) l2[2:2] ([])
  equal l1[3:4] (['cd']) l2[2:3] (['cd'])

```


## 7.Reference


本文參考於python官網的documentation

想要更深入了解 `difflib` 請看：

https://docs.python.org/2/library/difflib.html