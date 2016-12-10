---
layout: post
title: '機器學習 -- Logistic Regression Model (3D)'
date: 2014-03-15 01:47
comments: true
categories: [machine_learning]
---

## 1.Introduction

這次來講一下機器學習(Machine Learning)的Logistic Model


搭上3D的風潮,讓我們來看看3D的Logistic Model畫出來會是如何


至於機器學習（Machine Learning）是什麼？

簡而言之,Machine Learning是一種讓機器根據已知的data,預測出未知的data情形如何

Logistic Regression是一種機器學習的Model,可以用來處理分類問題

從input data的feature,可以判斷出output data該歸到那一類

Logistic Regression的model如下


$$

h(x)=\frac{1}{1+e^{-W^{T} X }}

$$


其中 $$h(x)$$ 是hypothesis,藉由hypothesis,可以用已知的data來預知未知的data

$$X$$是input data, $$W$$ 是weight,這兩者皆是矩陣

<!--more-->

$$

X=

\begin{bmatrix}

	x_{0}\\

  x_{1}\\

  x_{2}\\

\end{bmatrix},

W=

\begin{bmatrix}

	w_{0}\\

  w_{1}\\

  w_{2}\\

\end{bmatrix}, \mspace{20mu} W^{T} X =w_{0}x_{0}+w_{1}x_{1}+w_{2}x_{2} 

$$



用error function $$E_{in}$$ 的gradient,調整 $$W$$ ,可以讓h(x)根據data建立出一個的model,作出準確的預測


$$

\bigtriangledown E_{in}(W)=\frac{1}{1+e^{-y W^{T} X }} \cdot (-yX) \\

W\leftarrow  W-\eta \bigtriangledown E_{in}(W)

$$

其中,$$y$$ 是實際的結果,藉此可比對 $$h(x)$$ 預測出的結果,和實際結果是否相等

$$\eta$$ 是學習速度,可以調整演算法收斂的速度



在此不提詳細理論,

若想更深入了解,請看coursea機器學習的線上課程 


## 2. Draw 3D Logistic Model


接著來畫畫看3D的Logistic Model是什麼樣子

首先,開一個新的檔案 *plot_logistic.py* ,載入以下模組



```python plot_logistic.py
from mpl_toolkits.mplot3d import Axes3D
import matplotlib.pyplot as plt
import numpy as np

```


然後寫個function來畫圖



```python plot_logistic.py
def plot_logistic(w0,w1,w2):
    fig = plt.figure()
    ax = fig.gca(projection='3d')
    X1, X2 = np.mgrid[-2:2:0.25, -2:2:0.25]
    X0 = np.ones(X1.shape)
    Z=np.divide(1,1+np.exp(-1*(np.multiply(w0,X0)+np.multiply(w1,X1)+np.multiply(w2,X2)) ))
    surf = ax.plot_wireframe(X1, X2, Z, cstride=1, rstride=1)
    ax.set_title(("w0=%s, w1=%s, w2=%s")%(w0,w1,w2))
    plt.show()

```


這個function可以讓我們輸入不同的weight,畫出不一樣的圖

其中 `w0,w1,w2` 是weight,

`Z=np.divide(1,1+np.exp(np.multiply(w0,X0)+np.multiply(w1,X1)+np.multiply(w2,X2)))`

是logistic Model


接著到interactive mode來跑剛剛寫的function



```python

>>> import plot_logistic as pl

```


首先,用不同的w0畫畫看



```python

>>> pl.plot_logistic(0,1,1)
>>> pl.plot_logistic(2,1,1)
>>> pl.plot_logistic(-2,1,1)

```


<img src="http://lh3.googleusercontent.com/-jCPyrHFtLTw/UyP_03T4BkI/AAAAAAAAAcc/cPuCsRbEs2I/w480-h320-no/logistic1.png">

<img src="http://lh3.googleusercontent.com/-FihSXpokG6c/UyP_05hcLTI/AAAAAAAAAcY/iQ13UbUUFqA/w480-h320-no/logistic2.png">

<img src="http://lh4.googleusercontent.com/-IxWybG4zcBE/UyP_140FJYI/AAAAAAAAAc8/TdMEnA7m81A/w480-h320-no/logistic3.png">


再改變不同的w1來畫



```python

>>> pl.plot_logistic(0,1,1)
>>> pl.plot_logistic(0,2,1)
>>> pl.plot_logistic(0,5,1)

```


<img src="http://lh3.googleusercontent.com/-jCPyrHFtLTw/UyP_03T4BkI/AAAAAAAAAcc/cPuCsRbEs2I/w480-h320-no/logistic1.png">

<img src="http://lh5.googleusercontent.com/-ElJA7s-qR5w/UyP_1oAHn3I/AAAAAAAAAdA/33SZMCbcrUY/w480-h320-no/logistic4.png">

<img src="http://lh4.googleusercontent.com/-NGiKStaFfJE/UyP_1nbKXiI/AAAAAAAAAcs/oBSw0gc7djE/w480-h320-no/logistic5.png">


改變w2的結果和改變w1的結果呈對稱,在此省略


## 3. Learning by Logistic Model


接著來看看如何用logistic model來進行機器學習

開一個新的檔案 *logistic_model.py* ,並載入必要模組



```python logistic_model.py
from mpl_toolkits.mplot3d import Axes3D
import matplotlib.pyplot as plt 
import numpy as np

```


假設有這些data



```python logistic_model.py
X0=np.array([ 1.,  1.,  1.,  1.,  1.,  1.,  1.,  1.,  1.,  1.,  1.,  1.,  1.,
              1.,  1.])
X1=np.array([ 0.56084642,  1.3977301 ,  1.37559576,  1.25336531, -1.90734535,
              1.41981271,  0.96873065, -1.95225824, -0.65156755,  0.84391794,
              1.37463405, -1.90385285, -1.9961716 , -1.53774446,  0.62736126])

X2=np.array([-0.54631195,  1.9883947 , -1.77695809,  1.74944549, -0.99031191,
              1.6854447 , -1.78734606, -0.46925818, -1.72241882, -0.68654333,
              1.70881411,  1.22240194, -1.87704629,  1.10556042, -1.892201  ])

Y=np.array([-0.2041112 , -0.10358391, -2.69506218,  0.01788122,  4.42147236,
            -0.35981413, -1.88860354,  4.87603575,  1.39744193, -0.86841621,
            -0.25309822,  5.96338706,  3.9784108 ,  5.14938121, -1.27926322])

```


接著,寫出畫圖的function



```python logistic_model.py
def plot_data(filename='data1'):
    fig = plt.figure()
    ax = fig.gca(projection='3d')
    surf = ax.scatter(X1[Y>=0], X2[Y>=0], np.ones(X1[Y>=0].shape) ,c='b',marker='o')
    surf = ax.scatter(X1[Y<0], X2[Y<0], np.zeros(X1[Y<0].shape) ,c='r',marker='^')
    plt.show()

```


接著到interactive mode來跑剛剛寫的function



```python

>>> import logistic_model as lm
>>> lm.plot_data()

```


<img src="http://lh5.googleusercontent.com/-03OAk99aDVQ/UyQTVzt5JiI/AAAAAAAAAd8/uDW14eX_vi0/w480-h320-no/data1.png">


再來看看,把剛剛的model加進來看看會怎樣

在 *logistic_model.py* 加入一個function

和剛剛的類似,但這個function有加入logistic model



```python logistic_model.py
def plot_data_and_model(w0,w1,w2):
    fig = plt.figure()
    ax = fig.gca(projection='3d')
    surf = ax.scatter(X1[Y>=0], X2[Y>=0], np.ones(X1[Y>=0].shape) ,c='b',marker='o')
    surf = ax.scatter(X1[Y<0], X2[Y<0], np.zeros(X1[Y<0].shape) ,c='r',marker='^')
    ##---Logistic Model---- 
    GX1, GX2 = np.mgrid[-2:2:0.25, -2:2:0.25]
    GX0 = np.ones(GX1.shape)
    Z = np.divide(1,1+np.exp(-1*(np.multiply(w0,GX0)+np.multiply(w1,GX1)+np.multiply(w2,GX2))))
    surf = ax.plot_wireframe(GX1, GX2, Z, cstride=1, rstride=1)
    ##---------------------
    plt.show()

```


到interactive mode重新載入 *logistic_model.py* ,再畫畫看



```

>>> reload(lm)
<module 'logistic_model' from 'logistic_model.py'>

>>> lm.plot_data_and_model(1,1,1)

```


<img src="">


看起來,這個model跟data好像差得很遠

沒關係,我們用h(x)的gradient來調整w0,w1,w2

$$

\bigtriangledown E_{in}(W)=\frac{1}{1+e^{-y W^{T} X }} \cdot (-y X) \\

W\leftarrow  W-\eta \bigtriangledown E_{in}(W)

$$


接著我們要用以上公式來調整 $$W$$ ,達到最佳化

在 *logistic_model.py* 加入以下function



```python logistic_model.py
def learning_logistic(times):
    w0,w1,w2=1,1,1
    for i in range(times):
        eta = 0.1
        temp_err = np.divide(1,1+np.exp((Y)*(np.multiply(w0,X0)+np.multiply(w1,X1)+np.multiply(w2,X2))))
        e0 = eta*np.average(temp_err*(-Y*X0))
        e1 = eta*np.average(temp_err*(-Y*X1))
        e2 = eta*np.average(temp_err*(-Y*X2))
        w0 = w0-e0 
        w1 = w1-e1 
        w2 = w2-e2 
    plot_data_and_model(w0,w1,w2)

```


簡而言之,這個function用一個迴圈,逐次來調整w0,w1,w2,把error降低

times是這個迴圈要跑幾次,

eta是學習速率,如果eta越小,就越慢收斂

但eta太大,有可能會錯過最佳解


接著,重新載入修改好的程式

分別讓迴圈跑1次,10次到10000次來看看,結果如何



```python

>>> reload(ml)
<module 'logistic_model' from 'logistic_model.py'>

>>> ml.learning_logistic(1)
>>> ml.learning_logistic(10)
>>> ml.learning_logistic(100)
>>> ml.learning_logistic(1000)
>>> ml.learning_logistic(10000)

```


結果如下：

<img src="http://lh5.googleusercontent.com/-7zj7cOhJPEI/UyQUdiwAp9I/AAAAAAAAAf0/nwRWG2AQHi4/w480-h320-no/model_1.png">

<img src="http://lh4.googleusercontent.com/-NAOxS1hvShE/UyQUdmlxMRI/AAAAAAAAAgI/461egO-xJWw/w480-h320-no/model_10.png">

<img src="http://lh3.googleusercontent.com/-SdDWeaxWkwA/UyQUePGcZOI/AAAAAAAAAfw/Mmdzl_eWWlU/w480-h320-no/model_100.png">

<img src="http://lh4.googleusercontent.com/-BDGOyehak7A/UyQUeoJNGoI/AAAAAAAAAgE/pcR_NCORpd4/w480-h320-no/model_1000.png">

<img src="http://lh6.googleusercontent.com/-ZndSShsS7ZU/UyQUfHgvlpI/AAAAAAAAAgA/BEwdIT-QwVc/w480-h320-no/model_10000.png">


動畫版：

<img src="http://lh3.googleusercontent.com/-QBkEkySQAEI/UyVEwjNnd-I/AAAAAAAAAi8/kvoOH4-g514/w480-h320-no/model_motion.gif">


很明顯地,我們可以看到機器正在學習,依照data來把model做調整

model可以符合大部分的data都可以達到準確預測

但還是有些例外,例如,右上角的藍色的點,不符合model的預測

這種data不符合大多數data的情形,就可能noice了


## 4. Further Reading


以上是Logistic Model很簡單的實作


想要學習Machine Learning的理論與應用

請到coursera上線上課程

主要有以下兩門課,雖然難易度不同,但都講得很好


#### 1.Andrew Ng. *Machine Learning*

Andrew Y. Ng 是Stanford University的知名教授,也是Google Brain開發團隊的成員之一

這門課基本概念講得很清楚,基本的概念以及應用都有提到,淺顯易懂,適合一般大眾

https://www.coursera.org/course/ml


#### 2.林軒田 *機器學習基石 (Machine Learning Foundations)*

林軒田教授是台大資工系的教授,也曾經帶領學生參加資料探勘比賽KDDCUP獲得世界冠軍

這門課有很多數學理論推導,作業多,難度頗高,適合想要成為機器學習的高手的人

https://www.coursera.org/course/ntumlone


