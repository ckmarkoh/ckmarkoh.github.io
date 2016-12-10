---
layout: post
title: '自然語言處理 -- Log-Linear Model'
date: 2014-04-28 11:00
comments: true
categories: [natural_language_processing, log_linear, entropy, tagging, classifier, maximum_entropy]
---

## 1. Introduction


在機器學習中有一種用於分類的演算法, 叫作 *Logistic Regression* , 可以把東西分成兩類


而在自然語言處理的應用, 常常需要處理多類別的分類問題, 像是 *Part of speech Tagging* 就是把一個字詞分類到名詞, 動詞, 形容詞, 之類的問題


如果二元分類的 *Logistic Regression* , 推廣到多種類別分類, 就可以處理這種分類問題


首先, 把二元分類的 *Logistic Regression* 公式, 稍做調整, 如下  

 

$$

\begin{align}

&p(y=true|X) = \frac{1}{1+e^{-W \cdot X }} 

= \frac{ e^{\frac{W \cdot X}{2}} }{ e^{\frac{W \cdot X}{2}}+e^{\frac{-W \cdot X}{2}}  } \\[12pt]

&p(y=false|X) = \frac{e^{-W \cdot X }}{1+e^{-W \cdot X }} 

= \frac{ e^{\frac{-W \cdot X}{2}} }{ e^{\frac{W \cdot X}{2}}+e^{\frac{-W \cdot X}{2}}  } \\

\end{align}

$$


針對多類別的  *Logistic Regression* , 叫作 *Multinomial logistic regression* , 如果總共有 $$k$$ 的類別, 每個類別的 *label* 為 $$c_{i} , i \in k $$ , 則公式如下

<!--more-->

$$

\begin{align}

&p(y=c_{1}|X) = \frac{ e^{W_{c_{1}} \cdot X} }{ \sum_{i=1}^{k} e^{W_{c_{i}} \cdot X} } \\[12pt]

&p(y=c_{2}|X) = \frac{ e^{W_{c_{2}} \cdot X} }{ \sum_{i=1}^{k} e^{W_{c_{i}} \cdot X} } \\[12pt]

&...\\[12pt]

&p(y=c_{k}|X) = \frac{ e^{W_{c_{k}} \cdot X} }{ \sum_{i=1}^{k} e^{W_{c_{i}} \cdot X} } \\


\end{align}

$$


在自然語言處理中, 由於 *feature value* , 也就是 $$X$$ , 通常不是數字, 例如 *前面幾個字的 Tag* 之類的, 這時就要用 *feature function* 把 *feature value* 轉成數字


所謂的 *feature function* , 就像是一個檢查器, 去檢查 *input data* 是否滿足某個 *feature* , 滿足的話則輸出 *1* , 不滿足者輸出 *0* , 以下為一個*feature function* 的例子


$$

f_{j}(c,x) =

\begin{cases} 

1 & \text{if}\mspace{15mu} tag_{i-1} = VB \mspace{15mu}\text{and} \mspace{15mu} c=NN \\

0 & \text{otherwise}

\end{cases}

$$

其中 , $$tag_{i-1}$$ 是前一個字的 *Tag* , 而 $$c$$ 為這個字的類別, 如果這個字的類別是 $$NN$$ , 且前一個字的 *Tag* 為 $$VB$$ , 則 $$f_{j}=1$$ , 若不滿足這些條件, 則 $$f_{j}=0$$


加入 *feature function* 以後 , 原本的 $$W_{c_{i}} \cdot X$$ 變為  $$\sum_{i=0}^{N}w_{c_{i}}f_{i}(c,x)$$ ,  *Multinomial logistic regression* 的公式變為這樣, 也就是所謂的 *Log-Linear Model*

$$

p(y=c_{i}|X) = \frac{ e^{ \sum_{i=0}^{N}w_{c_{i}}f_{i}(c,x) } }{ \sum_{j=1}^{k} e^{\sum_{j=0}^{N}w_{c_{j}}f_{j}(c,x)} } \\[12pt]

$$


再來, 要怎麼訓練這個 *Model* 呢？

*Training* 是一個求最佳解的過程, 要找到一組 *Weight* 可以使得 $$\sum_{i}p(Y^{(i)} \mid X^{(i)})$$ 為最大值, 公式為


$$

\mathop{\arg\,\max}\limits_{w} \sum_{i} p(Y^{(i)} \mid X^{(i)})  


$$


由於有時 *feature function* 的數量會太多, 容易導致 *Overfitting* , 為了避免此現象, 所以會減掉 $$\alpha R(w)$$ 以進行 *Regularization*

$$

\mathop{\arg\,\max}\limits_{w} \sum_{i} p(Y^{(i)} \mid X^{(i)})  -\alpha R(w)


$$


另外, 由於此最佳化後產生的結果, 會有最大的 *Entropy* , 故 *Log-Linear Model* 又稱為 *Maxmum Entropy Model* , 在此做不推導, 欲知詳情請看 *Berger et al. (1996). A maximum entropy approach to natural language processing.*


## 2. Example


舉個例子, 如何用 *feature function* 算出 *Tagging* 的機率值


假設現在要對以下句子進行 *Part of Speech Tagging* , 現在已經進行到了 ***race*** 這個字

$$

\text{Secretariat/}NNP \text{ is/}BEZ \text{ expected/}VBN \text{ to/}TO \text{ race/}\textbf{??} \text{ tomorrow/} 

$$


總共用了以下六種 *feature function* 


$$

\begin{align}

&f_{1}(c,x) =

\begin{cases} 

1 & \text{if}\mspace{15mu} word_{i} = \text{'race'} \mspace{15mu}\text{and} \mspace{15mu} c=NN \\

0 & \text{otherwise} 

\end{cases} \\[12pt]


&f_{2}(c,x) =

\begin{cases} 

1 & \text{if}\mspace{15mu} tag_{i-1} = TO \mspace{15mu}\text{and} \mspace{15mu} c=VB \\

0 & \text{otherwise} 

\end{cases} \\[12pt]


&f_{3}(c,x) =

\begin{cases} 

1 & \text{if}\mspace{15mu} suffix(word_{i}) = \text{'ing'} \mspace{15mu}\text{and} \mspace{15mu} c=VBG \\

0 & \text{otherwise} 

\end{cases} \\[12pt]


&f_{4}(c,x) =

\begin{cases} 

1 & \text{if}\mspace{15mu} isLowerCase(word_{i}) \mspace{15mu}\text{and} \mspace{15mu} c=VB \\

0 & \text{otherwise} 

\end{cases} \\[12pt]


&f_{5}(c,x) =

\begin{cases} 

1 & \text{if}\mspace{15mu} word_{i} = \text{'race'} \mspace{15mu}\text{and} \mspace{15mu} c=VB \\

0 & \text{otherwise} 

\end{cases} \\[12pt]


&f_{6}(c,x) =

\begin{cases} 

1 & \text{if}\mspace{15mu} tag_{i-1} = TO \mspace{15mu}\text{and} \mspace{15mu} c=NN \\

0 & \text{otherwise} 

\end{cases} 

\end{align}

$$


現在要求 ***race*** 這個字的 *Tag* 是 *NN* 還是 *VB* , 代入以上六個 *feature function* , 得出結

果於下表


$$

    \begin{array}{|c|c|} 

    \hline

		   &   & f1 & f2 & f3 & f4 & f5 & f6 \\ \hline

    VB & f & 0  & 1  & 0  & 1  & 1  & 0  \\ \hline

    VB & w & 0  & 0.8& 0  &0.01& 0.1& 0  \\ \hline

    NN & f & 1  & 0  & 0  & 0  & 0  & 1  \\ \hline

    NN & w &0.8 & 0  & 0  & 0  & 0  &-1.3  \\ \hline

		\end{array}

$$


其中 $$f$$ 是 *feature function* 算出來的值, $$w$$ 是 *weight* , 這個值通常是針對 *Training Data* 做最佳化得出來的值, *weight* 越大則表示 *feature* 所占的比重越重


接著把 $$w_{i}f_{i}(c,x)$$ 的值帶入公式


$$

\begin{align}

&P(NN \mid x) = \frac{e^{0.8+(-1.3)}}{e^{0.8+(-1.3)}+e^{0.8+0.01+0.1}} = 0.2 \\[12pt]

&P(VB \mid x) = \frac{e^{0.8+0.01+0.1}}{e^{0.8+(-1.3)}+e^{0.8+0.01+0.1}} = 0.8 \\

\end{align}

$$


算出結果 $$0.8>0.2$$ , 所以 ***race*** 的 *Tag* 為 *VB*


## 3. Implementation


接著來實作用 *Log-Linear Mode* 進行 *Part of Speech Tagging*


這次要用 *python nltk*  的 `MaxentClassifier` 來實作


首先, 開一個新的檔案 *loglinear.py* 貼上以下程式碼



```python loglinear.py
import nltk
import operator

class LogLinearTagger(nltk.TaggerI):

    def __init__(self,training_corpus):
        self.classifier = None
        self.training_corpus = training_corpus
    
    def train(self):
        self.classifier = nltk.MaxentClassifier.train(
                    reduce(operator.add, 
                        map(lambda tagged_sent :
                            self.sent_to_feature(tagged_sent)
                            ,self.training_corpus)),algorithm='megam' )
    
    def sent_to_feature(self,tagged_sent):
        return  map(lambda (i, elem) : 
                        apply( lambda token , tag : 
                             (self.extract_features(token, i, tag), elem[1])
                            ,zip(*tagged_sent))
                        ,enumerate(tagged_sent))

    def tag_sentence(self, sentence_tag):
        if self.classifier == None:
            self.train()
        return apply (lambda sentence : 
                    zip(sentence,
                    reduce(lambda x,y:  
                        apply(operator.add,
                            [x,[self.classifier.classify(self.extract_features(sentence, y[0], x))]])
                        , enumerate(sentence), []))
                    ,[map(operator.itemgetter(0),sentence_tag)])

    def evaluate(self,test_sents):
        return apply(lambda result_list : 
                    sum(result_list)/float(len(result_list))
                    , [reduce(operator.add,
                        map(lambda line:
                            map(lambda tag : int(tag[0] == tag[1])
                                , zip(map(operator.itemgetter(1),line),
                                      map(operator.itemgetter(1),self.tag_sentence(line))))
                           ,test_sents))])

    def extract_features(self, sentence, i, history):
        features = {}
        features["this-word"] =  sentence[i]
        if i == 0:
            features["prev-tag"] = "<START>"
        else:
            features["prev-tag"] = history[i-1]
        return features     

```


其中, `extract_features` 是用於把 *input sentence* 的 *feature* 取出來,  例如這次用到的 *feature* 有目前這個字是什麼 `"this-word"` ,和前一個字的 *Tag* 是什麼 `"prev-tag"` 


取出 *feature* 後 , `MaxentClassifier` 會自動根據這些 *feature* 產生 *feature function* 


接下來到 *python* 的 *interactive mode* 載入檔案



```python

>>> from loglinear import LogLinearTagger

```


這次要用 *brown corpus* 的 *category* , `news` 的前 *100* 句來當作 *Tranining Data* ,第 *100~200* 句當作 *Test Data* , 先輸入以下程式碼



```python

>>> from nltk.corpus import brown
>>> brown_tagged_sents = brown.tagged_sents(categories='news')
>>> train_sents = brown_tagged_sents[:100]
>>> test_sents = brown_tagged_sents[100:200]

```


接著用 *Training Data* 建立一個 *LogLinearTagger* 的 *class*



```python

>>> classifier = LogLinearTagger(train_sents)

```


在開始訓練之前, 我們先挑其中的一句, 看一下格式, 是已經 *Tag* 好的句子 , 我們以 `train_sents[31]` 為例



```python

>>> train_sents[31]
[('His', 'PP$'), ('petition', 'NN'), ('charged', 'VBD'), ('mental', 'JJ'), \
('cruelty', 'NN'), ('.', '.')]

```


看一下這句可以產生出哪些 *Feature*



```python

>>> for x in classifier.sent_to_feature(train_sents[31]):
...     print x
... 
({'prev-tag': '<START>', 'this-word': 'His'}, 'PP$')
({'prev-tag': 'PP$', 'this-word': 'petition'}, 'NN')
({'prev-tag': 'NN', 'this-word': 'charged'}, 'VBD')
({'prev-tag': 'VBD', 'this-word': 'mental'}, 'JJ')
({'prev-tag': 'JJ', 'this-word': 'cruelty'}, 'NN')
({'prev-tag': 'NN', 'this-word': '.'}, '.')

```


例如第一個字, `'His'` , 它的 *feature* 有 `'prev-tag': '<START>'` 和 `'this-word': 'His'` , *Tag* 的結果為 

`'PP$'` , 由於第一個字前面已經沒有字了, 也沒有 *Tag* 了, 所以我們用 `<START>` 來表示


再來就是要訓練 *classifier*, 執行 `classifier.train()` 就可以開始訓練, 但要花一點時間



```python

>>> classifier.train()
Scanning file...2268 train, 0 dev, 0 test, reading...done
optimizing with lambda = 0
it 1   dw 5.348e-01 pp 4.19728e+00 er 0.79850
it 2   dw 3.179e+00 pp 3.32097e+00 er 0.82760
it 3   dw 1.037e+00 pp 2.92326e+00 er 0.67549
it 4   dw 9.602e-01 pp 2.72106e+00 er 0.63933
it 5   dw 1.345e+00 pp 2.41257e+00 er 0.54012
it 6   dw 1.378e+00 pp 2.16177e+00 er 0.46429
......

```


如果出現以下錯誤訊息, 表示你沒安裝 *megan*



```python
    raise LookupError('\n\n%s\n%s\n%s' % (div, msg, div))
LookupError: 

===========================================================================
NLTK was unable to find the megam file!
Use software specific configuration paramaters or set the MEGAM environment variable.

  For more information, on megam, see:
    <http://www.cs.utah.edu/~hal/megam/>
===========================================================================

```


請到 http://www.umiacs.umd.edu/~hal/megam/version0_91/ 下載 *megan*

 

如果你是 *linux* 的使用者, 可直接下載執行檔, 放到 `/home/xxxxxx/bin/` 資料夾 ( 若你是使用 *Mac* 或 *Window$* , 則需要下載 *source code* 自行編譯


或者你可以把 *loglinear.py* 中的 `MaxentClassifier` 的 ` algorithm='megam' ` 去掉 , 變成這樣


```python loglinear.py
        self.classifier = nltk.MaxentClassifier.train(
                    reduce(operator.add, 
                        map(lambda tagged_sent :
                            self.sent_to_feature(tagged_sent)
                            ,self.training_corpus)) )

```


但這會導致訓練速度變得很慢


訓練好之後, 可以用 *Test Data* 看看結果如何 , 先挑一句, 以 `test_sents[10]` 為例



```python

>>> test_sents[10]
[('``', '``'), ('You', 'PPSS'), ('take', 'VB'), ('out', 'RP'), ('of', 'IN'), \
('circulation', 'NN'), ('many', 'AP'), ('millions', 'NNS'), ('of', 'IN'), \
('dollars', 'NNS'), ("''", "''"), ('.', '.')]

```


把這個句子放到訓練好的 `classifier` , 用它來 *Tag* , 比較一下跟原本的 *tag* 有何不同 



```python

>>> classifier.tag_sentence(test_sents[10])
[('``', '``'), ('You', 'VB'), ('take', 'VB'), ('out', 'RP'), ('of', 'IN'), \
('circulation', 'JJ'), ('many', 'AP'), ('millions', 'NNS'), ('of', 'IN'), \
('dollars', 'JJ'), ("''", "''"), ('.', '.')]

```


先用肉眼觀察, 我們發現 `classifier` 所得出的 *Tag* 有些和原本的一樣, 有些不一樣, 表示 `classifier` 有些字 *Tag* 錯了


可以用程式來算準確率, 用 `classifier.evaluate` ,  但注意的是, *input argument* 不是 *sentence* , 而是 *list of sentence* , 所以 *input argument* 要用 `[test_sents[10]]` , 如下



```python

>>> classifier.evaluate([test_sents[10]])
0.75

```


算出來後準確度是 *0.75* , 也就是說有 *75%* 的 *Tag* 是正確的


再來把所有的 *Test Data* 都做 *Evaluation* 看看



```python

>>> classifier.evaluate(test_sents)
0.6910327241818954

```


得的準確率約為 *69.1%*


這樣的準確率不是很理想, 原因是因為 *100* 句的 *Training Data* 實在是太少了


有興趣者可以試試看, 取 *2000* 句的 *Training Data* , 準確度應該會大幅提昇, 但是要花很久的時間訓練



## 3. Furtuer Reading


本文參考至這本教科書

[Speech and Language Processing](http://www.amazon.com/Speech-Language-Processing-2nd-Edition/dp/0131873210)

以及台大資工系 陳信希教授的 自然語言處理 課程講義

