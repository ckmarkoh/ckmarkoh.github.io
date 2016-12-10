---
layout: post
title: 'Optimization Method -- Gradient Descent & AdaGrad'
date: 2015-12-23 17:14
comments: true
categories: [adagrad, optimization]
---

## Introduction

在機器學習的過程中，常需要將 *Cost Function* 的值減小，需由最佳化的方法來達成。本文介紹 *Gradient Descent* 和 *AdaGrad* 兩種常用的最佳化方法。


## Gradient Descent

*Gradient Descent* 的公式如下：


$$

\textbf{x}_{t+1} \leftarrow \textbf{x}_{t} - \eta \textbf{g}_{t}

$$


其中， $$\eta$$ 為 *Learning Rate* ， $$\textbf{x} $$ 為最佳化時要調整的參數， $$\textbf{g}$$ 為最佳化目標函數對 $$\textbf{x}$$ 的梯度。 $$\textbf{x}_{t}$$ 為調整之前的 $$\textbf{x} $$ ，$$\textbf{x}_{t+1}$$ 為調整之後的 $$\textbf{x} $$ 。


舉個例子，如果目標函數為 $$ f(x,y) = y^2 - x^2  $$ ，起始參數為 $$(x,y) = (0.001,4)$$ ，則畫出來的圖形如下圖，曲面為目標函數，紅色的點為起始參數：

![](/images/pic/pic_00126.png)


<!--more-->


可藉由改變 $$(x,y)$$ 來讓 $$f(x,y)$$ 的值減小。 *Gradient Descent* 所走的方向為梯度最陡的方向，若 $$eta=0.3$$ 則 ：

$$

\begin{align}

& 	x \leftarrow  x - \eta  \dfrac{\partial f(x,y)}{\partial x}  \\

&  y \leftarrow  y - \eta  \dfrac{\partial f(x,y)}{\partial y} \\

\end{align}

$$


求出微分後得：


$$

\begin{align}

& 	x \leftarrow  x - \eta  \times (-2x)  \\

&  y \leftarrow  y - \eta  \times 2y \\

\end{align}

$$



代入數值，得：


$$

\begin{align}

& x = 0.001 - 0.3 \times (-2) \times 0.001 = 0.0016 \\

& y = 4 - 0.3 \times 2 \times 4 = 1.6 \\

\end{align}

$$


更新完後的結果如下：

![](/images/pic/pic_00127.png)


從上圖可看出，紅點移動到比較低的地方，即 $$f(x,y)$$ 變小了。

經過了數次改變 $$(x,y)$$ 值的循環之後，$$f(x,y)$$ 的值會越變越小，紅點移動的路徑如下圖所示：

![](/images/pic/pic_00118.png)


動畫版：

![](/images/pic/pic_00119.gif)


從上圖可發現，紅色的點會卡在 $$(0,0)$$ 附近（也就是Saddle Point），過了一陣子後才會繼續往下滾。


## AdaGrad


*Gradient Descent* 的缺點有：

(1) *Learning Rate* 不會隨著時間而減少

(2) *Learning Rate* 在每個方向是固定的


以上的(1)會使得在越接近近目標函數最小值時，越容易走過頭，(2)則會容易卡在目標函數的Saddle Point。


因為 *Gradient Descent* 只考慮目前的 *Gradient* ，如果可以利用過去時間在各個方向的 *Gradient* ，來調整現在時間點在各個方向的 *Learning Rate* ，則可避免以上兩種情型發生。


*AdaGrad* 的公式如下：


$$

\begin{align}

& \textbf{G}_{t} = \sum_{n=0}^{t} \textbf{g}_{n}^{2} \\

& \textbf{x}_{t+1} \leftarrow \textbf{x}_{t} - \frac{\eta}{\sqrt{\textbf{G}_{t}}} \textbf{g}_{t} \\

\end{align}

$$


其中，$$ \textbf{G}_{t} $$ 為過去到現在所有時間點所有的 $$\textbf{g}$$ 的平方和。由於  $$\textbf{x}$$ ， $$\textbf{g}$$和 $$\textbf{G}$$ 皆為向量，設 $$x_{i}$$ ， $$g_{i}$$ 和 $$G_{i}$$ 各為其元素，則公式可寫成：


$$

\begin{align}

& G_{i,t} = \sum_{n=0}^{t} g_{i,n}^{2} \\

& x_{i,t+1} \leftarrow x_{i,t} - \frac{\eta}{\sqrt{G_{i,t}}} g_{i,t} \\

\end{align}

$$


這公式可修正以上兩個 *Gradient Descent* 的缺點：


1.若時間越久，則 *Gradient* 平方和越大，使得 *Learning Rate* 越小，這樣就可以讓 *Learning Rate* 隨著時間減少，而在接近目標函數的最小值時，比較不會走過頭。


2.若某方向從過去到現在時間點 *Gradient* 平方和越小，則 *Learning Rate* 要越大。（直覺上來講，過去時間點 *Gradient* 越小的方向，在未來可能越重要，這種概念有點類似[tf-idf](/blog/2014/04/14/natural-language-processing-tf-idf)，在越少文檔中出現的詞，可能越重要。）由於各方向的 *Learning Rate* 不同，比較不會卡在 *Saddle Point* 。


前述例子，起始參數為 $$(x,y) = (0.001,4)$$ ，則畫出來的圖形如下圖，曲面為目標函數，藍點為起始參數：

![](/images/pic/pic_00128.png)


用 *AdaGrad* 來更新 $$(x,y)$$ 的值，如下：


$$

\begin{align}

& 	x_{t+1} \leftarrow  x_{t} - \frac{\eta}{\sqrt{\sum_{n=0}^{t}  

(\dfrac{\partial f(x_{n},y_{n})}{\partial x_{n}} )^{2} }} 

\dfrac{\partial f(x_{t},y_{t})}{\partial x_{t}}  \\


&  y_{t+1} \leftarrow  y_{t} - \frac{\eta}{\sqrt{\sum_{n=0}^{t}  

(\dfrac{\partial f(x_{n},y_{n})}{\partial y_{n}} )^{2}  }} 

\dfrac{\partial f(x_{t},y_{t})}{\partial y_{t}} \\

\end{align}

$$


化簡後得：

$$

\begin{align}

& 	x_{t+1} \leftarrow  x_{t} - \frac{\eta}{\sqrt{\sum_{n=0}^{t}  

( -2x_{n} )^{2} }} 

( -2x_{t} ) \\


&  y_{t+1} \leftarrow  y_{t} - \frac{\eta}{\sqrt{\sum_{n=0}^{t}  

( 2y_{n} )^{2} }} 

( 2y_{t} ) \\

\end{align}

$$


由於 *AdaGrad* 的 *Learning Rate* 會隨時間減小，所以初始化時可以給它較大的值，此例中，設 $$\eta = 1.0$$

代入 $$(x,y)$$ 的數值，如下：


$$

\begin{align}

& x = 0.001 -  \frac{1.0}{\sqrt{  ( (-2) \times 0.001 )^2  }} \times (-2) \times 0.001 = 1.001 \\

& x = 4 -  \frac{1.0}{\sqrt{  ( 2 \times 4 )^2  }} \times 2 \times 4 = 3 \\

\end{align}

$$

更新圖上的藍點，如下圖：

![](/images/pic/pic_00129.png)


再往下走一步， $$(x,y)$$ 的值如下：

$$

\begin{align}

& x = 1.001 -  \frac{1.0}{\sqrt{  ( (-2) \times 0.001 )^2  + ( (-2) \times 1.001 )^2 }} \times (-2) \times 1.001 = 2.001 \\

& x = 3 -  \frac{1.0}{\sqrt{  ( 2 \times 4 )^2 +  ( 2 \times 3 )^2  }} \times 2 \times 3 = 2.4 \\

\end{align}

$$

更新圖上的藍點，如下圖：

![](/images/pic/pic_00130.png)



經過了數次改變 $$(x,y)$$ 值的循環之後，$$f(x,y)$$ 的值會越變越小，藍點移動的路徑如下圖所示：

![](/images/pic/pic_00123.png)



動畫版：

![](/images/pic/pic_00124.gif)


由此可以發現， *AdaGrad* 不會卡在 *Saddle Point* 。


將 *Gradient Descent* 和 *AdaGrad* 畫在同一張圖上，比較兩者差異：

![](/images/pic/pic_00125.gif)


## Implementation


再來進入實作的部分：


首先,開啟新的檔案 adagrad.py 並貼上以下程式碼



```python adagrad.py
from mpl_toolkits.mplot3d import Axes3D
from matplotlib import cm
import matplotlib.pyplot as plt
import numpy as np

def func(x,y):
  return (y**2-x**2)

def func_grad(x,y):
  return (-2*x, 2*y)

def plot_func(xt,yt,c='r'):
  fig = plt.figure()
  ax = fig.gca(projection='3d',
        elev=35., azim=-30)
  X, Y = np.meshgrid(np.arange(-5, 5, 0.25), np.arange(-5, 5, 0.25))
  Z = func(X,Y) 
  surf = ax.plot_surface(X, Y, Z, rstride=1, cstride=1, 
    cmap=cm.coolwarm, linewidth=0.1, alpha=0.3)
  ax.set_zlim(-50, 50)
  ax.scatter(xt, yt, func(xt,yt),c=c, marker='o' )
  ax.set_title("x=%.5f, y=%.5f, f(x,y)=%.5f"%(xt,yt,func(xt,yt))) 
  plt.show()
  plt.close()

def run_grad():
  xt = 0.001 
  yt = 4 
  eta = 0.3 
  plot_func(xt,yt,'r')
  for i in range(20):
    gx, gy = func_grad(xt, yt)
    xt = xt - eta*gx
    yt = yt - eta*gy
    if xt < -5 or yt < -5 or xt > 5 or yt > 5:
      break
    plot_func(xt,yt,'r')

def run_adagrad():
  xt = 0.001
  yt = 4 
  eta = 1.0 
  Gxt = 0
  Gyt = 0
  plot_func(xt,yt,'b')
  for i in range(20):
    gxt,gyt = func_grad(xt, yt)
    Gxt += gxt**2
    Gyt += gyt**2
    xt = xt - eta*(1./(Gxt**0.5))*gxt
    yt = yt - eta*(1./(Gyt**0.5))*gyt
    if xt < -5 or yt < -5 or xt > 5 or yt > 5:
      break
    plot_func(xt,yt,'b')


```


其中， `func(x,y)` 為目標函數，`func_grad(x,y)` 為目標函數的 *gradient* ，而 `plot_func(xt,yt,c='r')` 可畫出目標函數的曲面， `run_grad()` 用來執行 *Gradient Descent* ， `run_adagrad()` 用來執行 *AdaGrad* 。 `xt` 和 `yt` 對應到前例的 $$(x,y)$$ ，而 `eta` 為 *Learning Rate* 。 `for i in range(20)` 表示最多會跑20個迴圈，而 `if xt < -5 or yt < -5 or xt > 5 or yt > 5` 表示，如果 `xt` 和 `yt` 超出邊界，則會先結束迴圈。


到 python console 執行：



```sh

>>> import adagrad

```


執行 *Gradient Descent* ，指令如下：



```sh

>>> adagrad.run_grad()

```


則程式會逐一畫出整個過程：


![](/images/pic/pic_00126.png)


![](/images/pic/pic_00127.png)


以此類推


執行 *Adagrad* ，指令如下：




```sh

>>> adagrad.run_adagrad()

```


則程式會逐一畫出整個過程：


![](/images/pic/pic_00128.png)


![](/images/pic/pic_00129.png)


![](/images/pic/pic_00130.png)


以此類推



## Reference 


#### Notes on AdaGrad

http://www.ark.cs.cmu.edu/cdyer/adagrad.pdf


#### Visualizing Optimization Algos

http://imgur.com/a/Hqolp