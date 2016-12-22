---
layout: post
title: 'Hierarchical Probabilistic Neural Network Language Model (Hierarchical Softmax)'
date: 2015-05-23 15:33
comments: true
categories: ['Natural Language Processing','Neural Networks']
---

## Introduction


將類神經網路應用在自然語言處理領域的模型有[Neural Probabilistic Language Model(NPLM)](/blog/2015/05/15/neural-network-neural-probabilistic-language-model)，但在實際應用時，運算瓶頸在於 *output layer* 的神經元個數，等同於總字彙量 $$\mid V\mid $$ 。

 

每訓練一個字時，要讓 *output layer* 在那個字所對應的神經元輸出值為 $$1$$ ，而其他 $$\mid V\mid -1$$ 個神經元的輸出為 $$0$$ ， 這樣總共要計算 $$\mid V\mid $$ 次，會使得訓練變得沒效率。


若要減少於 *output layer* 的訓練時間，可以把 *output layer* 的字作分類階層，先判別輸出的字是屬於哪類，再判斷其子類別，最後再判斷是哪個字。 


## Hierarchical Softmax


給定訓練資料為$$X$$ ，輸出字的集合為 $$Y$$ 。當輸入的字串為 $$x$$ ，輸出的字為 $$y$$ 時，訓練的演算法要將機率 $$P (Y=y \mid  X=x) $$ 最佳化。


如果 $$Y$$ 有 *10000* 種字，若沒有分類階層，訓練時就要直接對 $$P (Y = y \mid  X = x) $$ 做計算，即是對這 *10000*種字做計算，使 $$y$$ 所對應的神經元輸出為 *1* ，其它 *9999* 個神經元輸出 *0* ，這樣要計算 *10000* 次，如下圖：


![](/images/pic/pic_00079.png)


若在訓練前，就事先把 $$Y$$ 中的字彙分類好，以 $$C(y)$$ 代表字 $$y$$ 的類別，則可以改成用以下機率做最佳化：


<!--more-->


$$

P (Y = y | X = x) =

P (Y = y | C = c(y), X) \times P (C = c(y) | X = x)


$$


根據以上公式，先判斷 $$y$$ 是在哪類，要算 $$P (C = c(y) \mid  X = x)$$ 的值，再判斷 $$y$$ 是 $$c(y)$$ 中的哪個字，要算 $$P (Y = y \mid  C = c(y), X) $$ 的值。


若把它們分成 *100* 類，則每個類別有 *100* 種字。訓練時需要分成兩步來計算，第一步先計算 $$P (C = c(y) \mid  X = x)$$ ，此時只要對 *100* 種分類做計算，使 $$C(y)$$ 所對應的神經元輸出 *1* ，其它 *99* 個神經元輸出 *0* 。第二步是算 $$P (Y = y \mid  C = c(y), X)$$，即是讓 $$C(y)$$ 底下對應到 $$y$$ 的神經元輸出 *1* ，其它*99* 個神經元輸出 *0* ，這樣只需要計算 *200* 次即可，所做的計算只有原本的 $$\frac{1}{50}$$ ，如下圖：


![](/images/pic/pic_00080.png)


更進一步地，可以將 *output layer* 分成更多層，也就是說，將它用成樹狀結構，每一個節點代表一次分類，這樣一層層分下去，直到葉子節點，可分出是哪個字彙。分越多層，訓練時要計算的次數就會越小，分到最細的情況下，就變成二元分類樹，這樣所需的計算總量只有原本的 $$\frac{log_{2}\mid V\mid }{\mid V\mid }$$ 倍。


根據這個二元分類樹，可以把 $$Y$$ 中的每一個字，對應到一個由 *0* 和 *1* 所組成的二元字串 $$(b_{1}(y), . . . b_{m}(y))$$ ，若字串中第一個數字為 *0* ，則表示這個字在第一層神經元所計算出的值為 *0*，以此類推。若要用這個二元分類樹計算 $$P (Y = y \mid  X = x) $$ 可用以下公式來計算：


$$

P( Y = y | X = x) =

\prod_{j=1}^{m}P(b_{j}(y) | b_{j-1}(y),b_{j-2}(y),...,b_{1}(y), X=x)

$$


其中， $$（b_{j−1}(y),b_{j−2}(y),...,b_{1}(y))$$ 為長度小於 $$m$$ 的二元字串，即是在二元分類樹中不是葉子也不是根的節點。


例如字串 $$( b_{1}=0, b_{2}=1 )$$即表示先從根節點往 *0* 的分支走到子節點，再從這個子節點走到 *1* 的分支的子節點，此子節點可用字串 *01* 來表示。若要計算以上公式的條件機率，方法為，先給這些子節點有它們自己的語意向量，再用 *NPLM* 把子節點的語意向量和 $$x$$ 當成輸入值一起輸入 *NPLM* 來計算。


例如，假設詞庫裡總共有 *8* 個字， 則 $$m=3$$ ，若字彙 $$y$$ 的二元字串值 $$b_{1} = 1, b_{2} =0, b_{3}=0 $$ ，則它在二元分類樹的位置如下圖所示：

![](/images/pic/pic_00081.png)


則 $$P( Y = y \mid  X = x) $$ 為：


$$

P(b_{1} = 1 | X=x) \times

P(b_{2} = 0 | b_{1} = 1, X=x) \times 

P(b_{3} = 0| b_{2} = 0, b_{1} = 1, X=x)

$$


訓練時，將訓練資料 $$x$$ 輸入到 *NPLM* 的類神經網路中，並以二元分類樹的節點來代替它的 *output layer* 。


首先，先從根節點的神經元開始訓練，這一步是要算 $$P(b_{1} = 1 \mid  , X=x) $$ 。由於 $$b_{1} = 1 $$，所以要以根節點的神經元輸出為 *1* 來調整 *NPLM* 模型的參數，如下圖：

![](/images/pic/pic_00082.png)


再來，從根節點往左走，選 *1* 分支的子神經元來訓練，這一步是要算 $$P(b_{2} = 0 \mid  b_{1} = 1, X=x) $$ 。因為這是基於 $$b_{1} = 1$$ 的條件機率，所以要用字串 *1* 所對應它的語意向量和 $$x$$ 一起輸入到 *NPLM* 。另外，由於 $$b_{2} = 0 $$，要以輸出為 *0* 來調整 *NPLM* 模型的參數，如下圖：

![](/images/pic/pic_00083.png)


然後，從剛剛那個神經元的節點開始，選 *0* 分支的子神經元來訓練，這一步是要算 $$P(b_{3} = 0\mid  b_{2} = 0, b_{1} = 1, X=x) $$ 。因為這是基於 $$b_{1} = 1 , b_{2} = 0 $$ 的條件機率，所以要先將字串 *10* 所對應的語意向量和 $$x$$ 一起輸入到 *NPLM* 。另外，由於 $$b_{3} = 0 $$，要以輸出為 *0* 來調整 *NPLM* 模型的參數，如下圖：


![](/images/pic/pic_00084.png)


最後，選擇 *0* 的分支往下走，即可走到 $$y$$ ，結束了針對 $$x$$ 這筆資料的訓練過程。


![](/images/pic/pic_00085.png)


## Hierarchical Clustering


在訓練之前，要先把字彙的階層分類給建立好。要建立這個分類階層，有很多種方式，例如，使用 *WordNet* 所建立的上下位關係，來建立階層，或者可以用 *Hierarchical Clustering* ，像是[Brown Clustering](/blog/2014/10/25/natural-language-processing-brown-clustering)之類的，自動從訓練資料中建立分類階層。


## Conclusion


用 *Hierarchical Softmax* 來置換掉原本 *NPLM* 的 *output layer* ，可以使得原本要計算 $$\mid V\mid $$ 次的訓練，縮減為 $$log_{2}\mid V\mid $$ 次，大幅提升了訓練速度。因此， *Hierarchical Softmax* 被日後眾多種類神經網路相關的模型所採用，包括近年來很熱門的 *word2vec* 也是。


## Reference


Frederic Morin , Yoshua Bengio. Hierarchical probabilistic neural network language model. 2005
