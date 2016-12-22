---
layout: post
title: 'AdaDelta'
date: 2016-02-08 16:13
comments: true
categories: ['Optimization Methods']
---

## AdaGrad


本文接續 [Optimization Method -- Gradient Descent & AdaGrad ](/blog/2015/12/23/optimization-method-adagrad)。所提到的 *AdaGrad* ，及改良它的方法 -- *AdaDelta* 。


在機器學習最佳化過程中，用 *AdaGrad* 可以隨著時間來縮小 *Learning Rage* ，以達到較好的收斂效果。*AdaGrad* 的公式如下：


$$

\begin{align}

& \textbf{G}_{t} = \sum_{n=0}^{t} \textbf{g}_{n}^{2} \\

& \textbf{x}_{t+1} = \textbf{x}_{t} - \frac{\eta}{\sqrt{\textbf{G}_{t}}} \textbf{g}_{t} \\

\end{align}

$$


不過， *AdaGrad* 有個缺點，由於 $$\textbf{g}_{n}^{2}$$ 恆為正，故 $$\textbf{G}_{t} $$ 只會隨著時間增加而遞增，所以 $$\frac{\eta}{\sqrt{\textbf{G}_{t}}} $$ 只會隨著時間增加而一直遞減，如果 *Learning Rate* $$\eta$$的值太小，則 *AdaGrad* 會較慢才收斂。


舉個例子，如果目標函數為 $$ f(x,y) = y^2 - x^2  $$ ，起始點為 $$(x,y) = (0.001,4)$$ ， *Learning Rate* $$\eta=0.5$$ ，則整個最佳化的過程如下圖，曲面為目標函數，紅色的點為 $$(x,y)$$ ：


![](/images/pic/pic_00157.png)


<!--more-->


動畫版：

![](/images/pic/pic_00158.gif)


從上圖來看，一開始紅色點的下降速度很快，但越後面則越慢。


為了解決此問題，在調整 *Learning Rate* 時，不要往前一直加到最初的時間點，而只要往前加到某段時間即可。


但如果要從某段時間點的 $$\textbf{g}_{t}$$ 開始累加，則需要儲存某個時間點之後開始的每個 $$\textbf{g}_{t}$$ ，這樣會造成記憶體的浪費。有種較簡便的做法，即是用衰減係數 $$\rho$$ ，將上一時間點的  $$\textbf{G}_{t-1}$$ 乘上 $$ \rho$$ ，如下：


$$

\begin{align}

& \textbf{G}_{t} = \rho \textbf{G}_{t-1} + (1 - \rho) \textbf{g}_{t}^{2} \\

& 0 < \rho < 1 \\

\end{align}

$$


藉由衰減係數 $$\rho$$ ，可讓較早期時間點累加的 $$\textbf{g}_{t}^{2}$$ 衰減至 0 ，因此，不會使得 *Learning Rate* 只隨著時間而一直遞減。


## Correct Units of ΔX


 *Adagrad* 還有另一個問題，就是 $$\textbf{x}$$ 的修正量-- $$\Delta{\textbf{x}}$$ 為 $$\frac{\eta}{\sqrt{\textbf{G}_{t}}} \textbf{g}_{t}$$ ，假設它如果有「單位」的話，它的單位會與 $$\textbf{x}$$ 不同。 因 $$\Delta{\textbf{x}}$$  的單位與 $$\textbf{g}$$ 的單位相同，而會和 $$\textbf{x}$$ 不同，因為：


$$

   \text{ units of }\Delta{\textbf{x}}  \propto  \text{ units of } \textbf{g} \propto  \dfrac{\partial f}{\partial x } \propto \frac{1}{  \text{ units of } \textbf{x} }

$$


註：在此假設 $$f$$ 無單位。


相較之下， [*Newton's Method*](/blog/2016/01/25/optimization-method-newton) 中， $$\Delta{\textbf{x}} =  \eta   \textbf{H}^{-1} \textbf{g}$$， $$\Delta{\textbf{x}}$$ 的單位與 $$\textbf{x}$$ 的單位相同，因為：


$$

   \text{ units of }\Delta{\textbf{x}}  \propto  \text{ units of } \textbf{H}^{-1} \textbf{g} \propto 

   \frac{

   \dfrac{\partial f}{\partial x }

   }

   {   \dfrac{\partial^{2} f}{\partial x^{2} }

   }

   \propto   \text{ units of } \textbf{x} 


$$


但 *Newton's Method* 的缺點是，二次微分 *Hessian* 矩陣的反矩陣 $$\textbf{H}^{-1}$$ ，計算時間複雜度太高。如果只是為了要單位相同，是沒必要這樣算。


想要簡易求出  $$\textbf{H}^{-1}$$ 的單位，稍微整理一下以上公式，得出：


$$

   \Delta{\textbf{x}}  \propto  \frac{\dfrac{\partial f}{\partial x } } {\dfrac{\partial^{2} f}{\partial x^{2}}} \Rightarrow  \textbf{H}^{-1} \propto \frac{1 } {\dfrac{\partial^{2} f}{\partial x^{2}}} 

     \propto

   

   \dfrac{\Delta{x}}{\dfrac{\partial f}{\partial x }} \propto  \dfrac{\Delta{x}}{\textbf{g}}  


$$


因此，若要簡易求出  $$\textbf{H}^{-1} $$ 的單位，只要算 $$\dfrac{\Delta{x}}{\textbf{g}}  $$ 即可。


註：如果看不懂這段在寫什麼，請參考[Matthew D. Zeiler. ADADELTA: AN ADAPTIVE LEARNING RATE METHOD.](http://arxiv.org/abs/1212.5701)



## AdaDelta


*AdaDelta* 解決了 *AdaGrad* 會發生的兩個問題：

(1) *Learning Rate* 只會隨著時間而一直遞減下去

(2) $$\Delta{\textbf{x}}$$ 與 $$\textbf{x}$$ 的單位不同


*AdaDelta* 的公式如下：


$$

\begin{align}

& \textbf{G}_{t} = \rho \textbf{G}_{t-1} + (1 - \rho) \textbf{g}_{t}^{2} \\


& \Delta \textbf{x}_{t} = - \frac{\sqrt{\textbf{D}_{t - 1}+\epsilon}}{\sqrt{\textbf{G}_{t}+\epsilon}} \textbf{g}_{t} \\


& \textbf{D}_{t} = \rho \textbf{D}_{t-1} + (1 - \rho) \Delta \textbf{x}_{t}^{2} \\


& \textbf{x}_{t+1} = \textbf{x}_{t} + \Delta{x}_{t} \\


\end{align}

$$


其中， $$\rho$$ 和  $$\epsilon$$ 為常數。 $$\rho$$ 的作用為「衰減係數」，而 $$\epsilon$$ 是為了避免 $$\frac{\sqrt{\textbf{D}_{t - 1}+\epsilon}}{\sqrt{\textbf{G}_{t}+\epsilon}}$$ 的分母為 0 。


此處的 $$ \textbf{G}_{t} $$ 有點類似 *AdaGrad* 裡面的  $$ \textbf{G}_{t} $$ ，但如前面所述，  *AdaDelta* 的不是直接把 $$\textbf{g}_{t}^2$$ 直接累加上去，而是藉由衰減係數 $$\rho$$ ，可讓較早期時間點累加的 $$\textbf{g}_{t}^{2}$$ 衰減至 0 ，因此，不會使得 *Learning Rate* 只隨著時間一直遞減下去。


而 $$\textbf{D}_{t}$$ 的作用，則是使 $$\Delta{\textbf{x}}$$ 與 $$\textbf{x}$$ 有相同的單位，因為 $$ \frac{\sqrt{\textbf{D}_{t - 1}+\epsilon}}{\sqrt{\textbf{G}_{t}+\epsilon}}$$ 與 $$\textbf{H}^{-1}$$ 具有相同單位，如下：


$$

 \frac{\sqrt{\textbf{D}_{t - 1}+\epsilon}}{\sqrt{\textbf{G}_{t}+\epsilon}} \propto  \dfrac{\Delta{x}}{\textbf{g}}  \propto  \textbf{H}^{-1}

$$


根據前一段的結果，若 $$\Delta{\textbf{x}}  \propto   \textbf{H}^{-1} \textbf{g}$$，則 $$\Delta{\textbf{x}}$$ 與 $$\textbf{x}$$ 的單位相同。


另外，$$\textbf{D}_{t}$$ 可累加過去時間點的 $$\Delta{\textbf{x}}$$ ，這樣所造成的效果，有點類似  [*Gradient Descent with Momentum*](/blog/2016/01/16/optimization-method-momentum) ，使得現在時間點的 $$\Delta{\textbf{x}}$$ ，具有過去時間點的動量。


實際帶數字進去算一次 *AdaDelta* 。舉前述例子，假設 $$ f(x,y) = y^2 - x^2  $$ ，起始參數為 $$(x,y) = (0.001,4)$$ ，則畫出來的圖形如下圖，藍色點為起始點位置：


![](/images/pic/pic_00161.png)


用 *AdaDelta* 最佳化方法，初始值設 $$\textbf{G}_{0} = [0,0 ]^{T}  $$ ， $$ \textbf{D}_{0} = [0,0 ]^{T} $$ ，設參數 $$\rho = 0.5$$ ， $$\epsilon = 0.1 $$ ，更新 $$x,y $$ 的值，如下，（註：以下的向量 $$\textbf{G}$$ 、 $$\textbf{D}$$ 、 $$\Delta \textbf{x}$$ 等等的加減乘除運算，皆為 *Element-wise Operation* ）：


$$

\begin{align}

& \textbf{g}_{1} = 

\begin{bmatrix}

-2x_{0} \\[0.3em]

2y_{0} \\[0.3em]

\end{bmatrix}

=

\begin{bmatrix}

-2 \times 0.001 \\[0.3em]

2 \times 4 \\[0.3em]

\end{bmatrix} 

=

\begin{bmatrix}

-0.002 \\[0.3em]

8 \\[0.3em]

\end{bmatrix} \\



& \textbf{G}_{1} = 0.5 

\begin{bmatrix}

0  \\[0.3em]

0  \\[0.3em]

\end{bmatrix}

+ (1-0.5 ) 

\begin{bmatrix}

(-0.002)^{2}  \\[0.3em]

8^{2}  \\[0.3em]

\end{bmatrix} 

=

\begin{bmatrix}

2 \times 10^{-6}  \\[0.3em]

32  \\[0.3em]

\end{bmatrix} \\


& \Delta \textbf{x}_{1} = - 

\frac{\sqrt{

\begin{bmatrix}

0   \\[0.3em]

0 \\[0.3em]

\end{bmatrix} 

+ 0.1}}

{\sqrt{

\begin{bmatrix}

2 \times 10^{-6}  \\[0.3em]

32 \\[0.3em]

\end{bmatrix} 

 + 0.1}}

\begin{bmatrix}

-0.002  \\[0.3em]

8  \\[0.3em]

\end{bmatrix} 

= 

\begin{bmatrix}

0.00199998 \\[0.3em]

-0.44651646 \\[0.3em]

\end{bmatrix} \\


& \textbf{D}_{1} = 0.5 

\begin{bmatrix}

0  \\[0.3em]

0  \\[0.3em]

\end{bmatrix}

+ (1-0.5 ) 

\begin{bmatrix}

0.00199998^{2}  \\[0.3em]

(-0.44651646)^{2}  \\[0.3em]

\end{bmatrix} 

=

\begin{bmatrix}

1.99996 \times 10^{-6}  \\[0.3em]

0.09968847  \\[0 .3em]

\end{bmatrix} \\


&

\begin{bmatrix}

x_{1} \\[0 .3em]

y_{1} \\[0 .3em]

\end{bmatrix}

=

\textbf{x}_{1} = 

\begin{bmatrix}

0.001 \\[0.3em]

4 \\[0 .3em]

\end{bmatrix}

+ 

\begin{bmatrix}

0.00199998 \\[0.3em]

-0.44651646 \\[0.3em]

\end{bmatrix}

= 

\begin{bmatrix} 

0.00299998 \\[0.3em]

3.55348354 \\[0.3em]

\end{bmatrix}

\end{align}


$$


更新 $$x,y$$ 的值， $$x,y = 0.00299998, 3.55348354 \approx 0.00300,3.55348  $$ ，如下圖：

![](/images/pic/pic_00160.png)


再往下走一步， 計算 $$x,y$$ 的值，如下：  


$$

\begin{align}

& \textbf{g}_{2} = 

\begin{bmatrix}

-2x_{1} \\[0.3em]

2y_{1} \\[0.3em]

\end{bmatrix}

=

\begin{bmatrix}

-2 \times 0.00299998 \\[0.3em]

2 \times 3.55348354 \\[0.3em]

\end{bmatrix} 

=

\begin{bmatrix}

-0.00599996 \\[0.3em]

7.10696708 \\[0.3em]

\end{bmatrix} \\



& \textbf{G}_{2} = 0.5 

\begin{bmatrix}

2 \times 10^{-6}  \\[0.3em]

32  \\[0.3em]

\end{bmatrix}

+ (1-0.5 ) 

\begin{bmatrix}

(-0.00599996)^{2}  \\[0.3em]

7.10696708^{2}  \\[0.3em]

\end{bmatrix} 

=

\begin{bmatrix}

1.89997600 \times 10^{-5}  \\[0.3em]

41.25449057  \\[0.3em]

\end{bmatrix} \\



& \Delta \textbf{x}_{2} = - 

\frac{\sqrt{

\begin{bmatrix}

1.99996 \times 10^{−6}   \\[0.3em]

0.09968847 \\[0.3em]

\end{bmatrix} 

+ 0.1}}

{\sqrt{

\begin{bmatrix}

1.89997600 \times 10^{-6}  \\[0.3em]

41.25449057 \\[0.3em]

\end{bmatrix} 

 + 0.1}}

\begin{bmatrix}

-0.00599996 \\[0.3em]

7.10696708 \\[0.3em]

\end{bmatrix} 

= 

\begin{bmatrix}

0.00599945 \\[0.3em]

-0.49385501\\[0.3em]

\end{bmatrix} \\


& \textbf{D}_{2} = 0.5 

\begin{bmatrix}

1.99996 \times 10^{−6}  \\[0.3em]

0.09968847  \\[0.3em]

\end{bmatrix}

+ (1-0.5 ) 

\begin{bmatrix}

0.00599945^{2}  \\[0.3em]

(-0.49385501)^{2}  \\[0.3em]

\end{bmatrix} 

=

\begin{bmatrix}

1.89966806 \times 10^{-5}  \\[0.3em]

0.17179062  \\[0 .3em]

\end{bmatrix} \\


&

\begin{bmatrix}

x_{2} \\[0 .3em]

y_{2} \\[0 .3em]

\end{bmatrix}

=

\textbf{x}_{2} = 

\begin{bmatrix}

0.00299998 \\[0.3em]

3.55348354 \\[0 .3em]

\end{bmatrix}

+ 

\begin{bmatrix}

0.00599945 \\[0.3em]

−0.49385501 \\[0.3em]

\end{bmatrix}

= 

\begin{bmatrix} 

0.00899943 \\[0.3em]

3.05962853 \\[0.3em]

\end{bmatrix}

\end{align}


$$



更新 $$x,y$$ 的值， $$x,y = 0.00899943, 3.05962853 \approx 0.00900,3.05963  $$ ，如下圖：

![](/images/pic/pic_00161.png)


重複以上循環，整個過程如下圖：


![](/images/pic/pic_00162.png)


動畫版：


![](/images/pic/pic_00163.gif)


將 *Gradient Descent* （綠） ， *AdaGrad* （紅） 和 *AdaDelta* （藍） 畫在同一張圖上比較看看： 


![](/images/pic/pic_00164.gif)


從上圖可看出， *AdaDelta* 的 *Learning Rate* 會隨著坡度而適度調整，不會一直遞減下去，也不會像 *Gradient Descent* 一樣，容易卡在 *saddle point* （請見[ Optimization Method -- Gradient Descent & AdaGrad ](/blog/2015/12/23/optimization-method-adagrad)）。


## Implementation


再來進入實作的部分

首先，開啟新的檔案 adadelta.py 並貼上以下程式碼：

 


```python adadelta.py
from mpl_toolkits.mplot3d import Axes3D
from matplotlib import cm
import matplotlib.pyplot as plt
import numpy as np
from math import sqrt

XT = 0.001
YT = 4

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


def run_adagrad():
  xt, yt = XT, YT
  eta = 0.5 
  Gxt, Gyt = 0, 0
  plot_func(xt,yt,'r')
  for i in range(20):
    gxt, gyt = func_grad(xt, yt)
    Gxt += gxt**2
    Gyt += gyt**2
    xt -= eta*(1./(Gxt**0.5))*gxt
    yt -= eta*(1./(Gyt**0.5))*gyt
    if xt < -5 or yt < -5 or xt > 5 or yt > 5:
      break
    plot_func(xt,yt,'r')

def run_adadelta():
  xt, yt = XT, YT
  epsilon = 0.1
  rho = 0.5
  Gxt, Gyt = 0., 0.
  Dxt, Dyt = 0., 0.
  plot_func(xt,yt,'b')
  for i in range(20):
    gxt, gyt = func_grad(xt, yt)
    Gxt, Gyt = rho * Gxt + (1-rho) * (gxt**2) , rho * Gyt + (1-rho) * (gyt**2)
    dxt, dyt  = -(sqrt(Dxt + epsilon) / sqrt(Gxt + epsilon)) * gxt , \
                -(sqrt(Dyt + epsilon) / sqrt(Gyt + epsilon)) * gyt
    Dxt, Dyt =  rho * Dxt + (1-rho) * (dxt**2) , rho * Dyt + (1-rho) * (dyt**2)
    xt += dxt
    yt += dyt
    if xt < -5 or yt < -5 or xt > 5 or yt > 5:
      break
    plot_func(xt,yt,'b')
 

```


其中， `func(x,y)` 為目標函數， `func_grad(x,y)` 為目標函數的 gradient ，而 `plot_func(xt,yt,c='r')` 可畫出目標函數的曲面， `run_adagrad()` 用來執行 *AdaGrad* ， `run_adadelta()` 用來執行 *AdaDelta* 。


到 python console 執行：



```sh

>>> import adadelta

```


執行 *AdaGrad* ，指令如下：



```sh

>>> adadelta.run_adagrad()

```


則程式會逐一畫出整個過程：


![](/images/pic/pic_00165.png)


![](/images/pic/pic_00166.png)


![](/images/pic/pic_00167.png)


以此類推


執行 *AdaDelta* ，指令如下：



```sh

>>> adadelta.run_adadelta()

```


則程式會逐一畫出整個過程：


![](/images/pic/pic_00168.png)


![](/images/pic/pic_00169.png)


![](/images/pic/pic_00170.png)


以此類推


## Reference


[Matthew D. Zeiler. ADADELTA: AN ADAPTIVE LEARNING RATE METHOD.](http://arxiv.org/abs/1212.5701)

[Visualizing Optimization Algos](http://imgur.com/a/Hqolp)
