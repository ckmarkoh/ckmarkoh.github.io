---
layout: post
title: '類神經網路 -- Recurrent Neural Network'
date: 2015-06-06 09:32
comments: true
categories: [neural_network, back_propagation]
---

## Introduction


在數位電路裡面，如果一個電路沒有 *latch* 或 *flip flop* 這類的元件，它的輸出值只會取決於目前的輸入值，和上個時間點的輸入值是無關的，這種的電路叫作 *combinational circuit* 。


對於類神經網路而言，如果它的值只是從輸入端一層層地依序傳到輸出端，不會再把值從輸出端傳回輸入端，這種神經元就相當於 *combinational circuit* ，也就是說它的輸出值只取決於目前時刻的輸入值，這樣的類神經網路稱為 *feedforward neural network* 。


如果一個電路有 *latch* 或 *flip flop* 這類的元件，它的輸出值就跟上個時間點的輸入值有關，這種的電路它稱為 *sequential circuit* 。


所謂的 *Recurrent Neural Network* ，是一種把輸出端再接回輸入端的類神經網路，這樣可以把上個時間點的輸出值再傳回來，記錄在神經元中，達成和 *latch* 類似的效果，使得下個時間點的輸出值，跟上個時間點有關，也就是說，這樣的神經網路是有 *記憶* 的。


## Recurrent Neural Network


由一個簡單神經元所構成的 *Recurrent Neural Network* ，構造如下：

![](/images/pic/pic_00095.png)


這個神經元在 $$t$$ 時間，訓練資料的輸入值為 $$x_{t}$$ ，訓練資料的答案為 $$y_{t}$$ ，神經元 $$n$$ 的輸出值 $$n_{out,t}$$ ，可用以下公式表示：


<!--more-->


$$

\begin{aligned}

& n_{in,t} = w_{c}x_{t}+ w_{p}n_{out,t-1} + w_{b} \\

& n_{out,t} = \frac{1}{1+e^{-n_{in,t}}} \\

\end{aligned}

$$

其中， $$n_{in,t}$$ 為輸入神經元 $$n$$ 的值， $$w_{c}$$ 是給目前的時間(current)時，輸入值 $$x_{t}$$ 的權重， $$w_{p}$$ 是給上個時間點(previous)時，輸出值 $$n_{out,t-1}$$ 的權重，而 $$w_{b}$$ 為 *bias* 。從上圖可看出，紫色的線將神經網路的輸出端 $$n_{out}$$ 連回輸入端 $$n_{in}$$ ，使得於時間 $$t$$ 的輸出值跟上個時間點 $$t-1$$ 的輸出值有關。


可以把這個神經元從時間點 $$0$$ 到時間點 $$t$$ 的運算，展開成下圖：


![](/images/pic/pic_00096.png)


從上圖，最左邊開始，依序將 $$x_{0},x_{1},...,x_{t}$$ 輸入神經元 $$n$$ ，而依序得出的值為 $$n_{out,0},n_{out,1},...,n_{out,t}$$ 。神經元 $$n$$ 在時間點 $$t-1$$ 的輸出值 $$n_{out,t-1}$$ ，會接到時間點 $$t$$ 時的輸入值 $$n_{in,t}$$ 。 


## Training Recurrent Neural Network


訓練 *recurrent neural network* 的方法，和訓練 *feedforward neural network* 的方法一樣，都可以用 *back propagation* 。但是在 *recurrent neural network* 中，要依據時間順序，將值從最後一個時間點，回傳到第一個時間點。


在時間點 $$t$$ 時的 *cost function* 為：


$$

J=−y_{t}log(n_{out,t})−(1−y_{t})log(1−n_{out,t})

$$


計算 *recurrent neural network* 的 *back propagation* 要分為兩部分來算，先算好時間點位於 $$t$$ 的偏微分 $$\dfrac{\partial J}{ \partial n_{in} } $$ 值，再依序往前算出時間點 $$t$$ 之前的偏微分值，如下：


$$

\dfrac{\partial J}{ \partial n_{in,s} } = 

\begin{cases} 

(\dfrac{\partial J}{ \partial n_{out,s} } )

(\dfrac{ \partial n_{out,s}}{\partial n_{in,s} } ) & \text{if } s = t \\

(\dfrac{\partial J}{ \partial n_{in,s+1} } )

(\dfrac{ \partial n_{in,s+1}}{\partial n_{out,s} } )

(\dfrac{ \partial n_{out,s}}{\partial n_{in,s} } )

  & \text{otherwise}

\end{cases}

$$


其中， $$s$$ 為 $$0$$ 到 $$t$$ 中的其中一個時間點。用 [Backward Propagation 詳細推導過程](/blog/2015/05/28/neural-network-backward-propagation) 所提到的推導方法，可推導出 $$ (\dfrac{\partial J}{ \partial n_{out,s} } )(\dfrac{ \partial n_{out,s}}{\partial n_{in,s} } )$$ 、 $$(\dfrac{ \partial n_{in,s+1}}{\partial n_{out,s} } )$$ 與 $$(\dfrac{ \partial n_{out,s}}{\partial n_{in,s} } )$$ 的值，並令 $$ \delta_{in,s} = \dfrac{\partial J}{ \partial n_{in,s} } $$ 代入以上公式，得出：


$$

\begin{align}

\delta_{in,s}  = 

\begin{cases} 

n_{out,s} - y_{s}  & \text{if } s = t \\

\delta_{in,s+1} w_{p} n_{out,s} (1-n_{out,s})  & \text{otherwise}

\end{cases}

\end{align}

$$


此公是可分為兩部分，當 $$s = t$$ 時，與  $$s\neq t$$ 時。計算 $$\delta_{s}$$ 的方式不同。

在 $$s = t$$ 時， $$\delta_{s}$$ 的傳遞過程就如同 *feedforward neural network* ，如下圖：

![](/images/pic/pic_00097.png)



若 $$s\neq t$$ 時， 要算 $$\delta_{in,s}$$ 之前，要先從 $$s+1$$ 時間點將 $$\delta_{in,s+1}$$ 傳遞過來，傳遞過程如下圖：


![](/images/pic/pic_00098.png)


因為需要把 $$\delta$$ 從後面的時間點往前面傳，故這個過程又稱為 *back propagation through time* 。

於時間點 $$s$$ 計算完 $$\delta$$ 後，用以下公式將 $$s$$ 時間點算出的偏微分值，更新到神經元的權重：


$$

\begin{align}

& w_{c} \leftarrow w_{c} - 

\eta \dfrac{\partial J}{ \partial n_{in,s} } \dfrac{\partial n_{in,s}}{\partial w_{c}} \\ 

& w_{b} \leftarrow w_{b} - 

\eta \dfrac{\partial J}{ \partial n_{in,s} } \dfrac{\partial n_{in,s}}{\partial w_{b}} \\

& w_{p} \leftarrow w_{p} - 

\eta \dfrac{\partial J}{ \partial n_{in,s} } \dfrac{\partial n_{in,s}}{\partial w_{p}}  \\

\end{align}

$$


用 [Backward Propagation 詳細推導過程](/blog/2015/05/28/neural-network-backward-propagation) ，求出 $$\dfrac{\partial n_{in,s}}{\partial w_{c}}$$ 、 $$\dfrac{\partial n_{in,s}}{\partial w_{b}} $$ 和 $$\dfrac{\partial n_{in,s}}{\partial w_{p}}$$ 的值，並換成用 $$\delta_{in,s}$$ 代替 $$\dfrac{\partial J}{ \partial n_{in,s} } $$ 代入以上公式，得出：


$$

\begin{align}

& w_{c} \leftarrow w_{c} - \eta \delta_{in,s} x_{s} \\ 

& w_{b} \leftarrow w_{b} - \eta \delta_{in,s} \\

& w_{p} \leftarrow w_{p} - \eta \delta_{in,s} n_{out,s-1} \\

\end{align}

$$


此過程如下圖所示：


![](/images/pic/pic_00099.png)


## Implementation


再來是實作的部分，以下是個簡單的應用，用 *Recurrent Neural Network* 來預測一個字串序列中，下一個可能出現的字是什麼。例如，給定以下字串：



```
001001001

```


根據這個字串的特徵，如果連續出現了兩個 *0* ，可以預測下個出現的為 *1* ，若前面兩個字為 *10* 則可預測下個出現的自為 *0* ，以此類推。


以下為實作部分：



```python simple_rnn.py
from math import e                                                                                    
from random import random

def sigmoid(x):
    return 1/float(1+e**(-1*x))

def my_rand():
    return random()-0.5

def rnn(x):
    r = 0.05
    w_p, w_c, w_b = my_rand(),my_rand(),my_rand()
    l = len(x)
    n_in = [0]*l
    n_out = [0]*l
    for h in range(10000):
        for i in range(l-1): 
            n_in[i] = w_c * x[i] + w_p * n_out[i] + w_b 
            n_out[i+1] = sigmoid(n_in[i])
        for i in range(l-1): 
            for j in range(i+1):
                k =  (i-j)
                if j == 0:
                    d_c = n_out[k+1] - x[k+1]
                else:
                    d_c = w_p * n_out[k+1] * (1-n_out[k+1]) * d_c
                w_c = w_c - r * d_c * x [k]
                w_b = w_b - r * d_c
                w_p = w_p - r * d_c * n_out[k]
    
    for i in range(l-1):
        n_in[i] = w_c * x[i] + w_p * n_out[i] + w_b
        n_out[i+1] = sigmoid(n_in[i])
    for w in zip(x,n_out):
        print w[0],w[1]


```


其中， `x` 為輸入的序列， `n_out` 為神經元預測的結果。進行這個演算法之前，首先，先給權重 `w_p, w_c, w_b` 的初始值用介於 *-0.5~0.5* 之間的隨機值。再來是進行訓練過程，用 *for loop* 進行了 *10000* 次的訓練，在每次的訓練過程中，先進行 *forward propagation* 依時間順序，算出每個時間點的 `n_out` 。再來是用 *back propagation through time* 來更新 `w_p, w_c, w_b` 的值。訓練完後，進行一次 *forward propogation* 用訓練過程得出的權重來預測序列的下一個字，並將預測結果印出。

 

到 *interactive mode* 執行以下程式，輸入序列 *001001001* 。



```

>>> import smallrnn
>>> smallrnn.rnn([0,0,1,0,0,1,0,0,1])
0 0
0 0.203697155215
1 0.93315712478
0 0.00354811782422
0 0.215230877663
1 0.945971073339
0 0.00455854449755
0 0.218600850027
1 0.949254861129

```


左側為輸入序列，右側為預測的結果，可以發現 *recurrent neural network* 可以預測出下個字可能會是 *0* 還是 *1* 。當左側為 *1* 時，右側的數字會接近於 *1* 。


## Further Reading


關於 *recurrent neural network* 可參考 coursera 課程 Geoffrey Hinton. Neural Networks for Machine Learning

https://www.coursera.org/course/neuralnets

