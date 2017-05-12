---
layout: post
title: "Torch NN Tutorial 6 : Backward Propagation ( part 3 : nn.Module ) "
date: 2017-01-01 18:36:08 +0800
comments: true
categories: ['Neural Networks','Torch']
---

## Backward Propagation in nn.Module

本文接續 [Torch NN Tutorial 4: Backward Propagation (part 1)](/blog/2017/01/01/torch-nn-tutorial-4-backward-propagation/) ，與 [Torch NN Tutorial 5: Backward Propagation (part 2)](/blog/2017/01/01/torch-nn-tutorial-5-backward-propagation/) 講解 `nn.Module` 中， 與 backward propagation 有關的程式碼。

`nn.Module` 程式碼：[https://github.com/torch/nn/blob/master/Module.lua](https://github.com/torch/nn/blob/master/Module.lua)

與 backward propagation 有關的，主要有以下三個運算 ：


$$
\begin{align}
&\text{1.} \mspace{20mu} \Delta_\textbf{W} = 0 \\
&   \mspace{40mu} \Delta_\textbf{b} = 0 \\
&\text{2.} \mspace{20mu} \Delta_\textbf{W} =  \frac{\partial J}{\partial \textbf{W}} =  \frac{\partial J}{\partial \textbf{y}}  \frac{\partial \textbf{y}}{\partial \textbf{W}}  \\ 
&   \mspace{40mu} \Delta_\textbf{b} =  \frac{\partial J}{\partial \textbf{b}} =  \frac{\partial J}{\partial \textbf{y}}  \frac{\partial \textbf{y}}{\partial \textbf{b}}  \\
&\text{3.} \mspace{20mu} \textbf{W} \leftarrow \textbf{W}  - \eta \Delta_\textbf{W} \\ 
&   \mspace{40mu} \textbf{b} \leftarrow \textbf{b} - \eta \Delta_\textbf{b} \\
\end{align}
$$

而這三個運算，分別對應到以下三個 function ：


1. zeroGradParameters

2. backward

3. updateParameters

本文將詳細講解這三個 function 的程式碼內容。

註：

1.本文的 $$\textbf{y}$$ 同前兩篇文（Torch NN Tutorial 4~5）中的 $$\bar{y}$$ 。

2.本文中的 $$\textbf{W}$$ 為矩陣，而 $$\textbf{x}$$ ，$$\textbf{y}$$ 為向量。

<!--more-->


## 1.zeroGradParameters

此步驟是將 $$\Delta_\textbf{W}$$ 和 $$\Delta_\textbf{b}$$ 歸零。

$$
\begin{align}
& \Delta_\textbf{W} = 0 \\
& \Delta_\textbf{b} = 0 \\
\end{align}
$$


在程式中， $$\Delta_\textbf{W}$$ 和 $$\Delta_\textbf{b}$$ 的變數是 `gradWeight` 和 `gradBias` 。可用 `zeroGradParameters` 將它們歸零，程式碼如下：


``` lua nn/Module.lua

function Module:zeroGradParameters()
   local _,gradParams = self:parameters()
   if gradParams then
      for i=1,#gradParams do
         gradParams[i]:zero()
      end
   end
end

```


在第五行中，可看到 `gradParams[i]:zero()` ，即是將所有的 `gradParams` ，包含 `gradWeight` 及 `gradBias` 歸零。

由於 `nn.Module` 本身沒有 `gradWeight` 和 `gradBias` ，如果要取得它們，就要透過 `self:parameters()` 來取得。
`parameters` 的程式碼如下：


``` lua nn/Module.lua

function Module:parameters()
   if self.weight and self.bias then
      return {self.weight, self.bias}, {self.gradWeight, self.gradBias}
   elseif self.weight then
      return {self.weight}, {self.gradWeight}
   elseif self.bias then
      return {self.bias}, {self.gradBias}
   else
      return
   end
end

```


從第二行開始，可看到，如果繼承 `nn.Module` 的類別有實作 `weight` ，則回傳 `weight` 和 `gradWeight` ，以此類推。

至於歸零的作用，因為在建立 Module 的時候， `gradWeight` 和 `gradBias` 並沒有被設為什麼值，它有可能是任意數。舉個例子，建立一個 `nn.Linear` ，命名為 `l1` ， input size 為 2， output size 為 3，一開始什麼都沒做，就直接印出  `gradWeight` 和 `gradBias` ，程式如下：


``` lua

l1 = nn.Linear(2,3)
print(l1.gradWeight)
print(l1.gradBias)

```


得出結果如下，它們可能是任意數：


``` sh

-1.2882e-231 -1.2882e-231
 2.1403e+161  3.9845e+252
  7.7075e-43   4.5713e-71
[torch.DoubleTensor of size 3x2]

-1.2882e-231
-1.2882e-231
  1.1659e-28
[torch.DoubleTensor of size 3]

```


如果用 `zeroGradParameters` 將它們歸零，程式碼如下：


``` lua

l1:zeroGradParameters()
print(l1.gradWeight)
print(l1.gradBias)

```


結果如下，可看到它們都歸零了：



``` sh

  0  0
  0  0
  0  0
[torch.DoubleTensor of size 3x2]

  0
  0
  0
[torch.DoubleTensor of size 3]

```


## 2.backward

再來看到 `backward` 的部分， `backward` 個功能由兩個 `function` 來執行，分別是 `updateGradInput` 及 `accGradParameters` 。


``` lua nn/Module.lua

function Module:backward(input, gradOutput, scale)
   scale = scale or 1
   self:updateGradInput(input, gradOutput)
   self:accGradParameters(input, gradOutput, scale)
   return self.gradInput
end

``` 


為何要分這兩部分？從數學公式上來看，如果 `input` 是 $$\textbf{x}$$ ， `output` 是 $$\textbf{y}$$ ，loss function 為 $$J$$   ，則：


$$
 \textbf{y}=\textbf{W}\textbf{x}+\textbf{b}
$$ 


`accGradParameters` 是負責計算以下兩項，也就是 `gradWeight` 和 `gradBias` 的值。


$$
\begin{align}
& \Delta_\textbf{W} =  \frac{\partial J}{\partial \textbf{W}} =  \frac{\partial J}{\partial \textbf{y}}  \frac{\partial \textbf{y}}{\partial \textbf{W}}  \\ 
& \Delta_\textbf{b} =  \frac{\partial J}{\partial \textbf{b}} =  \frac{\partial J}{\partial \textbf{y}}  \frac{\partial \textbf{y}}{\partial \textbf{b}}  \\
\end{align}
$$ 


除了以上兩項以外，如果 input 前面還有其他 layer 的話，就需計算 $$ \frac{\partial J}{\partial \textbf{x}}$$ ，並將此值往前傳遞。


`updateGradInput` 是負責計算 $$\frac{\partial J}{\partial \textbf{x}}$$ 的值，也就是 `gradInput` 的值，如下：


$$

\frac{\partial J}{\partial \textbf{x}} = \frac{\partial J}{\partial \textbf{y}}\frac{\partial \textbf{y}}{\partial \textbf{x}}

$$


在 `nn.Module` 中，這兩個 function 皆沒實作，需由繼承 `nn.Module` 的類別來實作。


``` lua nn/Module.lua

function Module:updateGradInput(input, gradOutput)
   return self.gradInput
end

function Module:accGradParameters(input, gradOutput, scale)
end

```


`nn.Linear` 則實作了這兩個 function。

`nn.Linear` 程式碼：[https://github.com/torch/nn/blob/master/Linear.lua](https://github.com/torch/nn/blob/master/Linear.lua)


### updateGradInput

先看 `updateGradInput` 的程式碼：


``` lua nn/Linear.lua

function Linear:updateGradInput(input, gradOutput)
   if self.gradInput then

      local nElement = self.gradInput:nElement()
      self.gradInput:resizeAs(input)
      if self.gradInput:nElement() ~= nElement then
         self.gradInput:zero()
      end
      if input:dim() == 1 then
         self.gradInput:addmv(0, 1, self.weight:t(), gradOutput)
      elseif input:dim() == 2 then
         self.gradInput:addmm(0, 1, gradOutput, self.weight)
      end

      return self.gradInput
   end
end

```




此處可分為兩部分來看，在 [Torch NN Tutorial 1 : NN.Module & NN.Linear](/blog/2016/12/19/torch-nn-tutorial-1-nn-module/) 曾講到，當 `input:dim() == 1` 時，是一次輸入一筆資料。

計算 `gradInput` 公式如下：

$$

\frac{\partial J}{\partial \textbf{x}} = \frac{\partial J}{\partial \textbf{y}}\frac{\partial \textbf{y}}{\partial \textbf{x}}

$$

公式中的 $$\frac{\partial J}{\partial \textbf{y}}$$ ，在程式中是 loss function 的 `gradInput` ，並從 `updateGradInput` 的參數 `gradOutput` 輸入。


假設 output size 為 3， input size 為 2，則在 forward propagation 時，$$\textbf{y}$$ 是由以下公式算出：


$$
\begin{align}
& \textbf{y} = \textbf{W}\textbf{x} + \textbf{b} \\
& \Rightarrow
\begin{bmatrix}
y_{1} \\
y_{2} \\ 
y_{3} \\
\end{bmatrix}
= 
\begin{bmatrix}
w_{11} & w_{12} \\
w_{21} & w_{22} \\
w_{31} & w_{32} \\
\end{bmatrix}
\begin{bmatrix}
x_{1}
x_{2}
\end{bmatrix}
+
\begin{bmatrix}
b_{1} \\
b_{2} \\
b_{3} \\
\end{bmatrix} \\
= 
&\begin{bmatrix}
w_{11}x_{1}+w_{12}x_{2}+b_{1} \\
w_{21}x_{1}+w_{22}x_{2}+b_{2} \\
w_{31}x_{1}+w_{32}x_{2}+b_{3} \\
\end{bmatrix} \\
\end{align}
$$


而 $$ \frac{\partial \textbf{y}}{\partial \textbf{x}} $$  的結果如下，


$$

\begin{align}
& \frac{\partial \textbf{y}}{\partial \textbf{x}} 
 = 
\begin{bmatrix}
\frac{\partial y_{1}}{\partial x_{1}} & \frac{\partial y_{1}}{\partial x_{2}} \\
\frac{\partial y_{2}}{\partial x_{1}} & \frac{\partial y_{2}}{\partial x_{2}} \\
\frac{\partial y_{3}}{\partial x_{1}} & \frac{\partial y_{3}}{\partial x_{2}} \\
\end{bmatrix} \\
&= 
\begin{bmatrix}
\frac{w_{11}x_{1}+w_{12}x_{2}+b_{1}}{\partial x_{1}} & \frac{w_{11}x_{1}+w_{12}x_{2}+b_{1}}{\partial x_{2}} \\
\frac{w_{21}x_{1}+w_{22}x_{2}+b_{2}}{\partial x_{1}} & \frac{w_{21}x_{1}+w_{22}x_{2}+b_{2}}{\partial x_{2}} \\
\frac{w_{31}x_{1}+w_{32}x_{2}+b_{3}}{\partial x_{1}} & \frac{w_{31}x_{1}+w_{32}x_{2}+b_{3}}{\partial x_{2}} \\
\end{bmatrix} \\
&= 
\begin{bmatrix}
w_{11} & w_{12} \\
w_{21} & w_{22} \\
w_{31} & w_{32} \\
\end{bmatrix}\\
&= \textbf{W}
\end{align}
$$

因此， $$\frac{\partial J}{\partial \textbf{x}}$$ 的值為：


$$
\begin{align}
&\frac{\partial J}{\partial \textbf{x}} 
=\begin{bmatrix}
\frac{\partial J}{\partial x_{1}} \\
\frac{\partial J}{\partial x_{2}}
\end{bmatrix} \\
&=\begin{bmatrix}
\frac{\partial J}{\partial y_{1}}\frac{\partial y_{1}}{\partial x_{1}} +
\frac{\partial J}{\partial y_{2}}\frac{\partial y_{2}}{\partial x_{1}} +
\frac{\partial J}{\partial y_{3}}\frac{\partial y_{3}}{\partial x_{1}} 
\\
\frac{\partial J}{\partial y_{1}}\frac{\partial y_{1}}{\partial x_{2}} +
\frac{\partial J}{\partial y_{2}}\frac{\partial y_{2}}{\partial x_{2}} +
\frac{\partial J}{\partial y_{3}}\frac{\partial y_{3}}{\partial x_{2}} 
\\
\end{bmatrix} \\

&=
\begin{bmatrix}
\frac{\partial y_{1}}{\partial x_{1}} & \frac{\partial y_{1}}{\partial x_{2}} \\
\frac{\partial y_{2}}{\partial x_{1}} & \frac{\partial y_{2}}{\partial x_{2}} \\
\frac{\partial y_{3}}{\partial x_{1}} & \frac{\partial y_{3}}{\partial x_{2}} \\
\end{bmatrix} ^{T}
\begin{bmatrix}
\frac{\partial J}{\partial y_{1}} \\
\frac{\partial J}{\partial y_{2}} \\ 
\frac{\partial J}{\partial y_{3}} \\
\end{bmatrix}\\

&=
\textbf{W}^{T}\frac{\partial J}{\partial \textbf{y}}
\end{align}\\
$$
  
從以上結果得知，要先將 `weight` 轉置，再和 `gradOutput` 做 矩陣-向量 乘積。如同程式碼中所寫 `self.gradInput:addmv(0, 1, self.weight:t(), gradOutput)`


如果 $$W$$ 和 $$\frac{\partial J}{\partial \textbf{y}}$$ 的值分別如下：


$$
\begin{align}
& \textbf{W} =
\begin{bmatrix}
 0.1453 & 0.5062 \\
 0.0635 & 0.4911 \\
-0.1080 & 0.1747 \\
\end{bmatrix} \\
& \frac{\partial J}{\partial \textbf{y}} = 
\begin{bmatrix}
1 \\
2 \\
3 \\ 
\end{bmatrix}
\end{align}
$$

則， $$\frac{\partial J}{\partial \textbf{x}}$$ 的值為：

$$
\begin{align}
&\frac{\partial J}{\partial \textbf{x}} 
=\textbf{W}^{T}\frac{\partial J}{\partial \textbf{y}}\\
&=
\begin{bmatrix}
 0.1453 & 0.0635 & -0.1080 \\ 
 0.5062 & 0.4911 & 0.1747 \\
 \end{bmatrix}
\begin{bmatrix}
1 \\
2 \\
3 \\ 
\end{bmatrix}\\
&=
\begin{bmatrix}
 0.1453 \times 1 & 0.0635 \times 2  & -0.1080 \times 3  \\ 
 0.5062 \times 1 & 0.4911 \times 2  & 0.1747 \times 3  \\
\end{bmatrix} \\
&=
\begin{bmatrix}
-0.0516 \\
 2.0126 \\
\end{bmatrix}\\
\end{align}
$$


執行 `updateGradInput`  ，程式碼如下：


``` lua

dj_dy = torch.Tensor{1,2,3}
x = torch.Tensor{4,5}
l1:updateGradInput( x,dj_dy )
print(l1.gradInput)

```


結果如下：


``` sh

-0.0516
 2.0126
[torch.DoubleTensor of size 2]

```


以上程式碼，輸入 `x` 只是為了滿足 `input:dim() == 1` 的條件， 而 `x` 的數值是不會影響結果的。


至於 `input:dim() == 2` 的情況，推導方式同上，在此不詳細推導。


再來看 `accGradParameters` 的程式碼：

### accGradParameters


``` lua nn/Linear.lua

function Linear:accGradParameters(input, gradOutput, scale)
   scale = scale or 1
   if input:dim() == 1 then
      self.gradWeight:addr(scale, gradOutput, input)
      if self.bias then self.gradBias:add(scale, gradOutput) end
   elseif input:dim() == 2 then
      self.gradWeight:addmm(scale, gradOutput:t(), input)
      if self.bias then
         -- update the size of addBuffer if the input is not the same size as the one we had in last updateGradInput
         updateAddBuffer(self, input)
         self.gradBias:addmv(scale, gradOutput:t(), self.addBuffer)
      end
   end
end

```

先看到 `input:dim() == 1` 的情形，計算 `gradWeight` 的公式如下：


$$

 \Delta_\textbf{W} =  \frac{\partial J}{\partial \textbf{W}} =  \frac{\partial J}{\partial \textbf{y}}  \frac{\partial \textbf{y}}{\partial \textbf{W}}  

$$

公式中的 $$\frac{\partial J}{\partial \textbf{y}}$$ ，在程式中是 loss function 的 `gradInput` ，並從 `updateGradInput` 的參數 `gradOutput` 輸入。



假設 output size 為 3， input size 為 2，則 $$ \frac{\partial \textbf{y}}{\partial \textbf{W}} $$  的結果如下，


$$

\begin{align}
& \frac{\partial \textbf{y}}{\partial \textbf{W}} 
 = 
\begin{bmatrix}
\frac{\partial y_{1}}{\partial w_{11}} & \frac{\partial y_{1}}{\partial w_{12}} \\
\frac{\partial y_{2}}{\partial w_{21}} & \frac{\partial y_{2}}{\partial w_{22}} \\
\frac{\partial y_{3}}{\partial w_{31}} & \frac{\partial y_{3}}{\partial w_{32}} \\
\end{bmatrix} \\
&= 
\begin{bmatrix}
\frac{w_{11}x_{1}+w_{12}x_{2}+b_{1}}{\partial w_{11}} & \frac{w_{11}x_{1}+w_{12}x_{2}+b_{1}}{\partial w_{12}} \\
\frac{w_{21}x_{1}+w_{22}x_{2}+b_{2}}{\partial w_{21}} & \frac{w_{21}x_{1}+w_{22}x_{2}+b_{2}}{\partial w_{22}} \\
\frac{w_{31}x_{1}+w_{32}x_{2}+b_{3}}{\partial w_{31}} & \frac{w_{31}x_{1}+w_{32}x_{2}+b_{3}}{\partial w_{32}} \\
\end{bmatrix} \\
&= 
\begin{bmatrix}
x_{1} & x_{2} \\
x_{1} & x_{2} \\
x_{1} & x_{2} \\
\end{bmatrix}\\
\end{align}
$$


因此， $$ \frac{\partial J}{\partial \textbf{W}} $$ 的值為：

$$
\begin{align}
& \frac{\partial J}{\partial \textbf{W}} 
 = 
\begin{bmatrix}
\frac{\partial J}{\partial y_{1}} \frac{\partial y_{1}}{\partial w_{11}} &
\frac{\partial J}{\partial y_{1}} \frac{\partial y_{1}}{\partial w_{12}} \\
\frac{\partial J}{\partial y_{2}} \frac{\partial y_{2}}{\partial w_{21}} &
\frac{\partial J}{\partial y_{2}} \frac{\partial y_{2}}{\partial w_{22}} \\
\frac{\partial J}{\partial y_{3}} \frac{\partial y_{3}}{\partial w_{31}} &
\frac{\partial J}{\partial y_{3}} \frac{\partial y_{3}}{\partial w_{32}} \\
\end{bmatrix} \\
&= 
\begin{bmatrix}
\frac{\partial J}{\partial y_{1}} x_{1} &
\frac{\partial J}{\partial y_{1}} x_{2} \\
\frac{\partial J}{\partial y_{2}} x_{1} &
\frac{\partial J}{\partial y_{2}} x_{2} \\
\frac{\partial J}{\partial y_{3}} x_{1} &
\frac{\partial J}{\partial y_{3}} x_{2} \\
\end{bmatrix} \\
&= 
\begin{bmatrix}
\frac{\partial J}{\partial y_{1}} \\
\frac{\partial J}{\partial y_{2}} \\
\frac{\partial J}{\partial y_{3}} \\
\end{bmatrix}
\begin{bmatrix}
x_{1} &  x_{2} \\
\end{bmatrix}\\
&= \frac{\partial J}{\partial \textbf{y}} \textbf{x}^{T}
\end{align}
$$


如果 $$\frac{\partial J}{\partial \textbf{y}}$$ 的值同前例， $$\textbf{x}$$ 的值如下：


$$
\textbf{x} =
\begin{bmatrix}
4 \\
5 \\
\end{bmatrix} 
$$


則， $$\frac{\partial J}{\partial \textbf{W}}$$ 的值為：


$$

\begin{align}
& \frac{\partial J}{\partial \textbf{W}} 
= \frac{\partial J}{\partial \textbf{y}}\textbf{x}^{T}\\
& = 
\begin{bmatrix}
1 \\
2 \\
3 \\ 
\end{bmatrix}
\begin{bmatrix}
4 & 5 
\end{bmatrix}\\ 
&=
\begin{bmatrix}
1 \times 4 & 1 \times 5 \\
2 \times 4 & 2 \times 5 \\
3 \times 4 & 3 \times 5 \\ 
\end{bmatrix}\\
&=
\begin{bmatrix}
4 & 5 \\
8 & 10 \\
12 & 15 \\ 
\end{bmatrix}\\
\end{align}
$$

計算 `gradBias` 的公式如下：


$$

 \Delta_\textbf{W} =  \frac{\partial J}{\partial \textbf{b}} =  \frac{\partial J}{\partial \textbf{y}}  \frac{\partial \textbf{y}}{\partial \textbf{b}}  

$$


而 $$ \frac{\partial \textbf{y}}{\partial \textbf{b}} $$  這項的結果如下，


$$
\begin{align}
& \frac{\partial \textbf{y}}{\partial \textbf{b}} 
= 
\begin{bmatrix}
\frac{\partial y_{1}}{\partial b_{1}} \\
\frac{\partial y_{2}}{\partial b_{2}} \\
\frac{\partial y_{3}}{\partial b_{3}} \\
\end{bmatrix} \\
&= 
\begin{bmatrix}
\frac{w_{11}x_{1}+w_{12}x_{2}+b_{1}}{\partial b_{1}} \\
\frac{w_{21}x_{1}+w_{22}x_{2}+b_{2}}{\partial b_{2}} \\
\frac{w_{31}x_{1}+w_{32}x_{2}+b_{3}}{\partial b_{3}} \\
\end{bmatrix} \\
&= 
\begin{bmatrix}
1 \\
1 \\
1 \\
\end{bmatrix}\\
\end{align}
$$

因此， $$ \frac{\partial J}{\partial \textbf{b}} $$ 的值為：

$$
\begin{align}
& \frac{\partial J}{\partial \textbf{b}} 
 = 
\begin{bmatrix}
\frac{\partial J}{\partial y_{1}} \frac{\partial y_{1}}{\partial b_{1}} \\
\frac{\partial J}{\partial y_{1}} \frac{\partial y_{1}}{\partial b_{2}} \\
\frac{\partial J}{\partial y_{2}} \frac{\partial y_{2}}{\partial b_{3}} \\
\end{bmatrix} \\
&= 
\begin{bmatrix}
\frac{\partial J}{\partial y_{1}} \\
\frac{\partial J}{\partial y_{2}} \\
\frac{\partial J}{\partial y_{3}} \\
\end{bmatrix} \\
&= \frac{\partial J}{\partial \textbf{y}} \\
\end{align}
$$

因此， $$ \frac{\partial J}{\partial \textbf{b}} $$ 的值，和 $$\frac{\partial J}{\partial \textbf{y}}$$ 的值相同。


執行 `accGradParameters`  ，程式碼如下：


``` lua

dj_dy = torch.Tensor{1,2,3}
x = torch.Tensor{4,5}
print( l1:accGradParameters( x,dj_dy ))
print(l1.gradWeight)
print(l1.gradBias)

```

結果如下：


``` sh

  4   5
  8  10
 12  15
[torch.DoubleTensor of size 3x2]

 1
 2
 3
[torch.DoubleTensor of size 3]

```

至於 `input:dim() == 2` 的情況，推導方式同上，在此不詳細推導。


## 3.updateParameters

此函數在 `nn.Module` 中即有實作其內容，它所進行的運算相當簡單，就是更新 $$\textbf{W}$$ 和 $$\textbf{b}$$ ：

$$
\begin{align}
& \textbf{W} \leftarrow \textbf{W}  - \eta \Delta_\textbf{W} \\ 
& \textbf{b} \leftarrow \textbf{b} - \eta \Delta_\textbf{b} \\
\end{align}
$$


程式碼如下：


``` lua  nn/Module.lua

function Module:updateParameters(learningRate)
   local params, gradParams = self:parameters()
   if params then
      for i=1,#params do
         params[i]:add(-learningRate, gradParams[i])
      end
   end
end

```

其中，公式中的 $$\eta$$ 即為程式碼中的 `learningRate` ，而程式中的第五行，則是負責將 `weight` 和 `bias` 更新。 其中， `weight` 和 `bias` 由第二行的 `parameters` 函數所取得，回傳到 `param` 中，而 `gradWeight` 和 `gradBias` 也用同樣方式取得，回傳到 `gradParams` 中。

假設 $$\eta = 0.02$$ ，$$\textbf{W} , \textbf{b} , \frac{\partial J}{\partial \textbf{W}} ,  \frac{\partial J}{\partial \textbf{b}}$$ 的值分別如下：


$$

\begin{align}
&\textbf{W}=
\begin{bmatrix}
 0.1453 & 0.5062 \\
 0.0635 & 0.4911 \\
-0.1080 & 0.1747 \\
\end{bmatrix} \\
&\textbf{b}=
\begin{bmatrix}
 0.2063 \\
-0.1635 \\
-0.0883 \\
\end{bmatrix} \\
& \frac{\partial J}{\partial \textbf{W}} = 
\begin{bmatrix}
4 & 5 \\
8 & 10 \\
12 & 15 \\ 
\end{bmatrix}\\
& \frac{\partial J}{\partial \textbf{b}} = 
\begin{bmatrix}
1 \\
2 \\
3 \\ 
\end{bmatrix}\\
\end{align}

$$

更新 $$\textbf{W}$$ 和 $$\textbf{b}$$ ：

$$
\begin{align}
& \textbf{W}  - \eta \Delta_\textbf{W} \\
& = 
\begin{bmatrix}
 0.1453 & 0.5062 \\
 0.0635 & 0.4911 \\
-0.1080 & 0.1747 \\
\end{bmatrix} -
0.02
\begin{bmatrix}
4 & 5 \\
8 & 10 \\
12 & 15 \\ 
\end{bmatrix}\\
& = 
\begin{bmatrix}
 0.0653 & 0.4062 \\
-0.0965 & 0.2911 \\
-0.3480 & -0.1253 \\
\end{bmatrix} \\

& \textbf{b}  - \eta \Delta_\textbf{b} \\
& = 
\begin{bmatrix}
 0.2063 \\
-0.1635 \\
-0.0883 \\
\end{bmatrix} -
0.02
\begin{bmatrix}
1 \\
2 \\
3 \\ 
\end{bmatrix}\\
&=
\begin{bmatrix}
 0.1863 \\
-0.2035 \\ 
-0.1483 \\
\end{bmatrix} 
\end{align}
$$


印出 `l1` 的 `weight` 和 `bias` ，執行 `updateParameters` ，比較執行前後的 `weight` 和 `bias` 差異，程式如下：


``` lua

print(l1.weight)
print(l1.bias)
l1:updateParameters(0.02)
print(l1.weight)
print(l1.bias)

```


結果如下：


``` sh

 0.1453  0.5062
 0.0635  0.4911
-0.1080  0.1747
[torch.DoubleTensor of size 3x2]

 0.2063
-0.1635
-0.0883
[torch.DoubleTensor of size 3]

 0.0653  0.4062
-0.0965  0.2911
-0.3480 -0.1253
[torch.DoubleTensor of size 3x2]

 0.1863
-0.2035
-0.1483
[torch.DoubleTensor of size 3]

```


## Materials

本次教學的完整程式碼於此：

[https://github.com/ckmarkoh/torch_nn_tutorials/blob/master/6_backward_propagation_part_3.ipynb
](https://github.com/ckmarkoh/torch_nn_tutorials/blob/master/6_backward_propagation_part_3.ipynb
)



