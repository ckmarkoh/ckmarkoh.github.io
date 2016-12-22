---
layout: post
title: 'Gradient Descent with Momentum'
date: 2016-01-16 08:01
comments: true
categories: ['Optimization Methods']
---

## Gradient Descent

在機器學習的過程中，常需要將 Cost Function 的值減小，通常用 Gradient Descent 來做最佳化的方法來達成。但是用 Gradient Descent 有其缺點，例如，很容易卡在 Local Minimum。


*Gradient Descent* 的公式如下：


$$

\textbf{x}_{t+1} \leftarrow \textbf{x}_{t} - \eta \textbf{g}_{t}

$$


關於Gradient Descent的公式解說，請參考：[Optimization Method -- Gradient Descent & AdaGrad](/blog/2015/12/23/optimization-method-adagrad)


## Getting Stuck in Local Minimum


舉個例子，如果 Cost Function 為 $$0.3y^{3}+y^{2}+0.3x^{3}+x^{2}$$ ，有 Local Minimum $$(x=0,y=0)$$ ，畫出來的圖形如下：

![](/images/pic/pic_00131.png)


<!--more-->


當執行 Gradient Descent 的時候，則會卡在 Local Minimum，如下圖：


![](/images/pic/pic_00132.gif)


解決卡在 Local Minimum 的方法，可加入 Momentum ，使它在 Gradient 等於零的時候，還可繼續前進。


## Gradient Descent with Momentum


Momentum 的概念如下： 當一顆球從斜坡上滾到平地時，球在平地仍會持續滾動，因為球具有動量，也就是說，它的速度跟上一個時間點的速度有關。


模擬 Momentum的方式很簡單，即是把上一個時間點用 Gradient 得出的變化量也考慮進去。


*Gradient Descent with Momentum* 的公式如下：


$$

\begin{align}

& \Delta_{\textbf{x},t+1 } \leftarrow  \beta \Delta_{\textbf{x},t } +  (1-\beta) \eta \textbf{g}_{t} 

& \text{, where }  0 <  \beta < 1 \\

\\

& \textbf{x}_{t+1} \leftarrow \textbf{x}_{t} - \Delta_{\textbf{x},t+1 } 

\end{align}

$$


其中 $$ \Delta_{\textbf{x},t +1} $$ 為 $$t+1$$ 時間點，修正 $$\textbf{x}$$ 值所用的變化量，而 $$\Delta_{\textbf{x},t }$$ 則是 $$t$$ 時間點的修正量，而 $$ \beta $$ 則是用來控制在 $$t+1$$ 時間點中的 $$\Delta_{\textbf{x},t+1}$$ 具有上個時間點的 $$\Delta_{\textbf{x},t}$$ 值的比例。 好比說，在 $$t+1$$ 時間點時，球的速度會跟 $$t$$ 時間點有關。 而 $$(1-\beta)$$ ，則是 $$t+1$$ 時間點算出之 Gradient $$\textbf{g}_{t}$$ 乘上 Learning Rate $$\eta$$ 後，在 $$\Delta_{\textbf{x},t+1}$$ 中所占的比例。


舉前述例子，若起始參數為 $$(x=3,y=3)$$  ，畫出目標函數，藍點為起始點 $$(x,y)$$ 的位置：

![](/images/pic/pic_00141.png)


用 Gradient Descent with Momentum 來更新 $$x,y$$ 的值，如下：


$$

\begin{align}

& \Delta_{x,t+1 } \leftarrow  \beta \Delta_{x,t } +  (1-\beta) \eta \dfrac{\partial f(x_{t},y_{t})}{\partial x_{t}} \\

& \Delta_{y,t+1 } \leftarrow  \beta \Delta_{y,t } +  (1-\beta) \eta \dfrac{\partial f(x_{t},y_{t})}{\partial y_{t}} \\

& x_{t+1} \leftarrow x_{t} - \Delta_{x,t+1 }  \\

& y_{t+1} \leftarrow y_{t} - \Delta_{y,t+1 } 


\end{align}

$$


化減後得：


$$

\begin{align}

& \Delta_{x,t+1 } \leftarrow  \beta \Delta_{x,t } +  (1-\beta) \eta (0.9 x_{t}^{2} + 2x_{t} ) \\

& \Delta_{y,t+1 } \leftarrow  \beta \Delta_{y,t } +  (1-\beta) \eta (0.9 y_{t}^{2} + 2y_{t} ) \\

& x_{t+1} \leftarrow x_{t} - \Delta_{x,t+1 }  \\

& y_{t+1} \leftarrow y_{t} - \Delta_{y,t+1 } 


\end{align}

$$



設初始化值 $$  \Delta_{x} = 0,  \Delta_{y} = 0 $$ ，參數 $$\beta = 0.9, \eta = 0.2 $$ ，代入 $$ x=3,y=3 $$ ，則：

$$

\begin{align}

& \Delta_{x,1 } =  0.9 \times  0 +  (1-0.9)\times 0.2 \times (0.9\times 3^{2}+2\times 3) = 0.282 \\

& \Delta_{y,1 } =  0.9 \times  0 +  (1-0.9)\times 0.2 \times (0.9\times 3^{2}+2\times 3) = 0.282 \\

& x_{1} = 3 - \Delta_{x,1 } = 3 - 0.282 = 2.718 \\

& y_{1} = 3 - \Delta_{y,1 } = 3 - 0.282 = 2.718

\end{align}

$$


更新圖上的藍點，如下圖：

![](/images/pic/pic_00142.png)


再往下走一步， $$ x,y $$  的值如下：


$$

\begin{align}

& \Delta_{x,2 } =  0.9 \times  0.282 +  (1-0.9)\times 0.2 \times (0.9\times 2.718^{2}+2\times 2.718) = 0.4955 \\

& \Delta_{y,2 } =  0.9 \times  0.282 +  (1-0.9)\times 0.2 \times (0.9\times 2.718^{2}+2\times 2.718) = 0.4955\\

& x_{2} = 3 - \Delta_{x,2 } = 2.718  - 0.4955 = 2.2225 \\

& y_{2} = 3 - \Delta_{y,2 } = 2.718  - 0.4955 = 2.2225

\end{align}

$$


更新圖上的藍點，如下圖：

![](/images/pic/pic_00143.png)


在以上兩步中，可發現 $$ \Delta_{x }, \Delta_{y }$$ 的值逐漸變大。由於一開始 $$ \Delta_{x }, \Delta_{y }$$ 都是零，它會跟前一個時間點的值有關，所以看起來就好像是球從斜坡上滾下來時，慢慢加速，而在球經過 Local Minimum時，也會慢慢減速，不會直接卡在 Local Minimum 。整個過程如下圖：

![](/images/pic/pic_00136.png)


動畫版：

![](/images/pic/pic_00137.gif)



## Implementation


再來進入實作的部分

首先，開啟新的檔案 momentum.py 並貼上以下程式碼：



```python momentum.py
from mpl_toolkits.mplot3d import Axes3D
from matplotlib import cm
import matplotlib.pyplot as plt
import numpy as np

def func(x,y):
  return (0.3*y**3+y**2+0.3*x**3+x**2)

def func_grad(x,y):
  return (0.9*x**2+2*x, 0.9*y**2+2*y )

def plot_func(xt,yt,c='r'):
  fig = plt.figure()
  ax = fig.gca(projection='3d',
        elev=7., azim=-175)
  X, Y = np.meshgrid(np.arange(-5, 5, 0.25), np.arange(-5, 5, 0.25))
  Z = func(X,Y) 
  surf = ax.plot_surface(X, Y, Z, rstride=1, cstride=1, 
    cmap=cm.coolwarm, linewidth=0.1, alpha=0.3)
  ax.set_zlim(-20, 100)
  ax.scatter(xt, yt, func(xt,yt),c=c, marker='o' )
  ax.set_title("x=%.5f, y=%.5f, f(x,y)=%.5f"%(xt,yt,func(xt,yt))) 
  plt.show()
  plt.close()

def run_grad():
  xt = 3 
  yt = 3 
  eta = 0.1
  plot_func(xt,yt,'r')
  for i in range(20):
    gxt, gyt = func_grad(xt,yt)
    xt = xt - eta * gxt
    yt = yt - eta * gyt
    if xt < -5 or yt < -5 or xt > 5 or yt > 5:
      break
    plot_func(xt,yt,'r')

def run_momentum():
  xt = 3 
  yt = 3 
  eta = 0.2
  beta = 0.9
  plot_func(xt,yt,'b')
  delta_x = 0
  delta_y = 0
  for i in range(20):
    gxt, gyt = func_grad(xt,yt)
    delta_x = beta * delta_x + (1-beta)*eta*gxt
    delta_y = beta * delta_y + (1-beta)*eta*gyt
    xt = xt - delta_x
    yt = yt - delta_y 
    if xt < -5 or yt < -5 or xt > 5 or yt > 5:
      break
    plot_func(xt,yt,'b')

```


其中， `func(x,y)` 為目標函數，`func_grad(x,y)` 為目標函數的 *gradient* ，而 `plot_func(xt,yt,c='r')` 可畫出目標函數的曲面， `run_grad()` 用來執行 *Gradient Descent* ， `run_momentum()` 用來執行 *Gradient Descent with Momentum* 。 `xt` 和 `yt` 對應到前例的 $$(x,y)$$ ，而 `eta` 為 *Learning Rate* 。 `for i in range(20)` 表示最多會跑20個迴圈，而 `if xt < -5 or yt < -5 or xt > 5 or yt > 5` 表示，如果 `xt` 和 `yt` 超出邊界，則會先結束迴圈。



到 python console 執行：



```sh

>>> import momentum

```


執行 *Gradient Descent* ，指令如下：



```sh

>>> momentum.run_grad()

```


則程式會逐一畫出整個過程：


![](/images/pic/pic_00138.png)


![](/images/pic/pic_00139.png)


![](/images/pic/pic_00140.png)


以此類推


執行 *Gradient Descent with Momentum* ，指令如下：



```sh

>>> momentum.run_momentum()

```


則程式會逐一畫出整個過程：


![](/images/pic/pic_00141.png)


![](/images/pic/pic_00142.png)


![](/images/pic/pic_00143.png)


以此類推


## Reference 


#### Visualizing Optimization Algos

http://imgur.com/a/Hqolp
