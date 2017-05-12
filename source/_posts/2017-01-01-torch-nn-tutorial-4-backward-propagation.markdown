---
layout: post
title: "Torch NN Tutorial 4: Backward Propagation ( part 1 : Overview & Linear Regression )"
date: 2017-01-01 14:00:00 +0800
comments: true
categories: ['Neural Networks','Torch']
---


## Introduction

本文以 Linear Regression 為例，介紹 Torch nn 如何進行 Backward Propagation。


Linear Regression 是以機器學習的方式，學出以下函數：


$$

y = wx+b

$$


以 Gradient Descent 的方式來進行 Linear regression 的流程如下：

$$
\begin{align}
&\text{1.} \mspace{20mu} \text{initialize } w \text{ and } b \\
&\text{2.} \mspace{20mu} \text{for i=0 ; i < max_epoch; i++} \\
&\text{3.} \mspace{40mu} \bar{y} = wx+b \\
&\text{4.} \mspace{40mu} J = (\bar{y} - y )^2 \\
&\text{5.} \mspace{40mu} \Delta_w = 0 \\
&   \mspace{60mu} \Delta_b = 0 \\
&\text{6.} \mspace{40mu} \frac{\partial J}{\partial \bar{y}} = 2(\bar{y} - y) \\
&\text{7.} \mspace{40mu} \Delta_w =  \frac{\partial J}{\partial w} =  \frac{\partial J}{\partial \bar{y}}  \frac{\partial y}{\partial w} =  \frac{\partial J}{\partial \bar{y}} \times x  \\ 
&   \mspace{60mu} \Delta_b =  \frac{\partial J}{\partial b} =  \frac{\partial J}{\partial \bar{y}}  \frac{\partial y}{\partial b}  = \frac{\partial J}{\partial \bar{y}} \times 1 \\
&\text{8.} \mspace{40mu} w \leftarrow w  - \eta \Delta_w \\ 
&   \mspace{60mu} b \leftarrow b - \eta \Delta_b \\
\end{align}
$$


其中 $$x$$ 為 training data， $$y$$ 為 golden value ， $$\bar{y}$$ 為 predicted value， $$w$$ 和 $$b$$ 分別為 weight 和 bias 。 max_epoch 為 for 迴圈執行次數。

以下詳細講解整個流程，並實作之。


<!--more-->

## Linear Regression by Gradient Descent


首先，載入 `nn` 套件，並產生 Training Data $$x$$ 及 $$y$$ ：



``` lua

require 'nn'
x = torch.linspace(1,3,3):resize(3,1)
y = x*2+1
print(x)
print(y)

```


以上程式，假設 $$y$$ 是由 $$y = 2 x + 1 $$ 產生出來的。而訓練的目標，是要讓 $$w=2, b=1$$ 。


產生出來的 `x` 為 `[1,2,3]` ， `y` 為 `[3,5,7]` 如下：


``` sh

 1
 2
 3
[torch.DoubleTensor of size 3x1]

 3
 5
 7
[torch.DoubleTensor of size 3x1]

```


為了方便以 batch 計算，將 `x` 及 `y` 調整成 3x1 的大小。


建立完 training data 之後，可以開始進行 Linear Regression。


第一步，將 $$w$$ 和 $$b$$ 以隨機值初始化。


$$ \text{1.}  \mspace{20mu} \text{initialize } w \text{ and } b $$



建立 `nn.Linear` ，命名為 `l1` ，如下： 


``` lua

l1 = nn.Linear(1,1)
print(l1.weight)
print(l1.bias)

```


一開始， `weight` 和 `bias` 會以隨機值初始化，結果如下：


``` sh

 0.2055
[torch.DoubleTensor of size 1x1]

 0.7159
[torch.DoubleTensor of size 1]

```


再來是建立 loss function，命名為 `c1` ：


``` sh

c1 = nn.MSECriterion()

```


由於 loss function 為 Mimimum Square Error，所以 criterion 採用 `nn.MSECriterion`



再來，這裡先講解 for 迴圈中進行的運算。




第三步算 linear 的 forward propagation ：


$$\text{3.} \mspace{20mu} \bar{y} = wx+b $$

將 $$x,w,b$$ 的數值帶入以上公式，得出：

$$
\begin{align}
& \bar{y} = wx+b\\
& =
0.2055
\begin{bmatrix}
1 \\
2 \\
3 \\
\end{bmatrix}
+ 0.7159 \\
& =
\begin{bmatrix}
 0.2055 \times 1 + 0.7159 \\
 0.2055 \times 2 + 0.7159 \\
 0.2055 \times 3 + 0.7159 \\
\end{bmatrix} \\
& =
\begin{bmatrix}
 0.9214 \\
 1.1269 \\
 1.3325 \\
\end{bmatrix}
\end{align}
$$


程式碼如下：


``` lua

y_ = l1:forward(x)
print(y_)

```


結果如下：


``` sh

 0.9214
 1.1269
 1.3325
[torch.DoubleTensor of size 3x1]

```


第四步，計算 coss function 的 forward propagation ：



$$\text{4.} \mspace{20mu} J = (\bar{y} - y )^2 $$

將 $$y,\bar{y}$$ 的數值帶入以上公式，得出：

$$
\begin{align}
& J = (\bar{y} - y)^2\\
& =
\begin{bmatrix}
(0.9214 - 3)^2 \\
(1.1269 - 5)^2 \\
(1.3325 - 7)^2 \\
\end{bmatrix} \\
& =
\begin{bmatrix}
 4.3206 \\
 15.0009 \\
 32.1206 \\
\end{bmatrix}
\end{align}
$$

如果輸入的值有多個維度，則 loss function 會將個維度的值，加起來平均，結果如下：

$$

J = \frac{4.3206 + 15.0009 + 32.1206}{3} = 17.147313377985 

$$


計算 $$J$$ 值的程式碼如下：


``` lua

j = c1:forward(y_,y)
print(j)

```


loss 值 `j` 如下：


``` sh

17.147313377985 

```


再來，要進行 backward propagation。


第五步，先將 $$ \Delta_w $$ 和 $$ \Delta_b $$ 歸零：

$$
\begin{align}
&\text{5.} \mspace{40mu} \Delta_w = 0 \\
&   \mspace{60mu} \Delta_b = 0 \\
\end{align}
$$


$$ \Delta_w $$ 和 $$ \Delta_b $$ 在程式中所對應的值，是 `l1` 的 `gradWeight` 和 `gradBias` 。在還沒歸零之前，這兩變數可能是任意值，印出這兩數的值，程式如下：



``` lua

print(l1.gradWeight)
print(l1.gradBias)

```


結果如下：


``` sh

1e-154 *
 -1.4917
[torch.DoubleTensor of size 1x1]

1e-154 *
-1.4917
[torch.DoubleTensor of size 1]

```


可用函式 `zeroGradParameters` 將這兩個值歸零，程式如下：


``` lua

l1:zeroGradParameters()
print(l1.gradWeight)
print(l1.gradBias)

```


結果為0，表示已歸零：


``` sh

 0
[torch.DoubleTensor of size 1x1]

 0
[torch.DoubleTensor of size 1]

```


第六步，計算 $$\frac{\partial J}{\partial \bar{y}}$$ 的值。


$$ \text{6.} \mspace{40mu} \frac{\partial J}{\partial \bar{y}} = 2(\bar{y} - y) $$


將 $$y,\bar{y}$$ 的數值帶入以上公式，得出：


$$
\begin{align}
& \frac{\partial J}{\partial \bar{y}} = 2(\bar{y} - y)\\
& =
\begin{bmatrix}
2(0.9214 - 3) \\
2(1.1269 - 5) \\
2(1.3325 - 7) \\
\end{bmatrix} \\
& =
\begin{bmatrix}
-1.3857 \\
-2.5820 \\
-3.7784 \\
\end{bmatrix}
\end{align}
$$

由於 $$\bar{y}$$ 有三筆資料，每筆資料都會各算出一個微分結果。
程式中，計算此值的方式即是呼叫 `c1` 的 `backward` ，如下：


``` lua

dj_dy_ =c1:backward(y_,y)
print(dj_dy_)

```


得出的結果即是 $$\frac{\partial J}{\partial \bar{y} } $$ ，結果如下：


``` sh 

-1.3857
-2.5820
-3.7784
[torch.DoubleTensor of size 3x1]

```


第七步，計算 $$ \Delta_w $$ 和 $$ \Delta_b $$ 的值：


$$
\begin{align}
&\text{7.} \mspace{40mu} \Delta_w =  \frac{\partial J}{\partial w} =  \frac{\partial J}{\partial \bar{y}}  \frac{\partial y}{\partial w} =  \frac{\partial J}{\partial \bar{y}} \times x  \\ 
&   \mspace{60mu} \Delta_b =  \frac{\partial J}{\partial b} =  \frac{\partial J}{\partial \bar{y}}  \frac{\partial y}{\partial b}  = \frac{\partial J}{\partial \bar{y}} \times 1 \\
\end{align}
$$


先看 $$\Delta_w $$ 的數值，將 $$x$$ 的值，以及先前算出的 $$ \frac{\partial J}{\partial \bar{y}}$$ 值代入，即可算出它：


$$
\begin{align}
& \Delta_w =  \frac{\partial J}{\partial \bar{y}} \times x \\
&=
\begin{bmatrix}
-1.3857 \times 1\\
-2.5820 \times 2\\
-3.7784 \times 3\\
\end{bmatrix}  \\
&=
\begin{bmatrix}
-1.3857 \\
-5.1640 \\
-11.3352 \\
\end{bmatrix}
\end{align}
$$


如果輸值為有多筆資料，則 $$\Delta_w$$ 的最終結果會將每筆的結果累加起來，如下：


$$
\Delta_w = -1.3857 + -5.1640 + -11.3352 = -17.8849
$$


而 $$\Delta_b$$ 的值是把 $$ \frac{\partial J}{\partial \bar{y}}$$ 中的每筆資料結果累積起來。

$$
\Delta_b = -1.3857 + -2.5820 + -3.7784 = -7.7461
$$



以程式來計算此兩值。呼叫 `l1` 的 `backward` ，輸入 $$\frac{\partial J}{\partial \bar{y} }$$ 到 `backward` 後，它與  $$\frac{\partial \bar{y}}{\partial w}$$ 和 $$\frac{\partial \bar{y}}{\partial b}$$ 相乘後結果分別為 $$\Delta_w$$ 和 $$\Delta_b$$ ，此兩數分別儲存於 `gradWeight` 和 `gradBias` 。


``` lua

l1:backward(x, dj_dy_ )
print(l1.gradWeight)
print(l1.gradBias)

```


結果如下：


``` sh

-17.8849
[torch.DoubleTensor of size 1x1]

-7.7461
[torch.DoubleTensor of size 1]

```


第八步，更新 weight 和 bias ，公式如下：


$$

\begin{align}
&\text{8.} \mspace{40mu} w \leftarrow w  - \eta \Delta_w \\ 
&   \mspace{60mu} b \leftarrow b - \eta \Delta_b \\
\end{align}

$$


其中， $$eta$$  是指 learning rate 。令 $$eta=0.2$$ ，更新 $$w$$ 和 $$b$$ ，數值如下：


$$
\begin{align}
& w  - \eta \Delta_w 
= 0.2055 - 0.02 \times (- 17.8849) = 0.5632 \\
& b - \eta \Delta_b 
= 0.7159 - 0.02 \times (-7.7461) = 0.8708 \\
\end{align}
$$


程式中，更新 `l1` 的 `weight` 和 `bias` 可以用 `updateParameters` ，它的輸入即是 $$eta$$ 。令 $$eta=0.2$$ ，程式如下：


``` lua

l1:updateParameters(0.02)
print(l1.weight)
print(l1.bias)

```


更新完後印出 `weight` 和 `bias` 的值，結果如下：


``` sh

 0.5632
[torch.DoubleTensor of size 1x1]

 0.8708
[torch.DoubleTensor of size 1]

```


把第三到第八步的 for 迴圈，串連起來。此 for 迴圈連續跑 500 次，每 50 次印出一次結果，程式碼如下：


``` lua

for i = 1,500 do
    y_ = l1:forward(x)
    j = c1:forward(y_,y)
    l1:zeroGradParameters()
    dj_dy_ =c1:backward(y_,y) 
    l1:backward(x,dj_dy_)
    l1:updateParameters(0.02)
    if i%50 == 0 then
        print("i:" .. tostring(i)
           .. " loss:" .. tostring(j) 
           .. " weight:" .. tostring(l1.weight[1][1])
           .. " bias:" .. tostring(l1.bias[1]) .. "\n")
    end
end

```


執行此程式，在訓練完 500 次以後，`loss` 會接近 0 ，而 `weight` 和 `bias` 會接近 2 和 1 了。結果如下：


``` sh

i:50 loss:0.015879173659737 weight:1.8543434015475 bias:1.3310995564975
  

i:100 loss:0.0098066687579948 weight:1.885537390716 bias:1.2602004152583
  

...  

i:450 loss:0.00033602903432248 weight:1.9788119352928 bias:1.0481654513272
  

i:500 loss:0.00020752499774989 weight:1.9833490852405 bias:1.0378514430405
  

```  


## Materials

本次教學的完整程式碼於此：

[https://github.com/ckmarkoh/torch_nn_tutorials/blob/master/4_backward_propagation_part_1.ipynb
](https://github.com/ckmarkoh/torch_nn_tutorials/blob/master/4_backward_propagation_part_1.ipynb
)





