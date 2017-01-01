---
layout: post
title: "torch nn tutorial 6 backward propagation"
date: 2017-01-01 14:36:08 +0800
comments: true
categories: 
published: false 
---

## Backward Propagation in nn.Module


此部分講解 `nn.Module` 中， 與 `backward` 有關的程式碼。

此部分講解以下三個 function ：

1. zeroGradParameters

2. backward

3. updateParameters


`nn.Module` 程式碼：[https://github.com/torch/nn/blob/master/Module.lua](https://github.com/torch/nn/blob/master/Module.lua)


### 1.zeroGradParameters

前文已提到，在計算 `nn.Module` 的 `backward` 之前，要先用 `zeroGradParameters` 將參數歸零，歸零的程式碼如下：


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
由於 `nn.Module` 本身沒有 `gradWeight` 和 `gradBias` 這兩個 variable ，所以要透過 `self:parameters()` 來取得。
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


如果繼承 `nn.Module` 的類別有實作 `weight` ，則回傳 `gradWeight` ，以此類推。

至於為何要歸零？ 因為在建立 Module 的時候， `gradWeight` 和 `gradBias` 並沒有被設為什麼值，它有可能是任意數，或者，`gradWeight` 和 `gradBias` 還保留著上一次運算的值。

舉個例子，建立一個 `nn.Linear` ，命名為 `l2` ， input 有二維， output 有三維，一開始什麼都沒做，就直接印出  `gradWeight` 和 `gradBias` ，程式如下：


``` lua

l2 = nn.Linear(2,3)
print(l2.gradWeight)
print(l2.gradBias)

```


得出結果如下，它們可能是任意數：


``` sh

-0.0000e+00 -0.0000e+00
5.2308e-143  1.2596e-71
 2.4420e+25  5.8979e-62
[torch.DoubleTensor of size 3x2]

 0
 0
 0
[torch.DoubleTensor of size 3]

```


如果用 `zeroGradParameters` 將它們歸零，程式碼如下：


``` lua

l2:zeroGradParameters()
print(l2.gradWeight)
print(l2.gradBias)

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


### 2.backward

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


則 backward propagation 需要計算以下三項：


$$
\begin{align}
& \frac{\partial J}{\partial \textbf{W}} = \frac{\partial J}{\partial \textbf{y}}\frac{\partial \textbf{y}}{\partial \textbf{W}}  \\  
& \frac{\partial J}{\partial \textbf{b}} = \frac{\partial J}{\partial \textbf{y}}\frac{\partial \textbf{y}}{\partial \textbf{b}} \\
& \frac{\partial J}{\partial \textbf{x}} = \frac{\partial J}{\partial \textbf{y}}\frac{\partial \textbf{y}}{\partial \textbf{x}} \\
\end{align}
$$


`updateGradInput` 是負責計算 $$\frac{\partial J}{\partial \textbf{x}}$$ 的值，也就是 `gradInput` 的值。

`accGradParameters` 是負責計算 $$\frac{\partial J}{\partial \textbf{W}}$$ 和 $$\frac{\partial J}{\partial \textbf{b}}$$ 的值，也就是 `gradWeight` 和 `gradBias` 的值。


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


先看 `updateGraadInput` 的程式碼：


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




此處可分為兩部分來看，在 [Torch NN Tutorial 1 : NN.Module & NN.Linear](/blog/2016/12/19/torch-nn-tutorial-1-nn-module/) 曾講到，當 `input:dim() == 1` 時，則是一次輸入一筆資料

假設 `loss function` 為 J ，則此時 `gradInput` 的算法如下：


$$

\frac{\partial J}{\partial \textbf{x}} = \frac{\partial J}{\partial \textbf{y}}\frac{\partial \textbf{y}}{\partial \textbf{x}}

$$

而 $$\frac{\partial J}{\partial \textbf{y}}$$ 是由 loss function 的 `updateGradInput` 算出的值所傳遞過來的，此值輸入 `gradOutput` 。


假設 `output` 有三維， `input` 有二維，則在 forward propagation 時，$$\textbf{y}$$ 是由以下公式算出：


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
&\frac{\partial J}{\partial \textbf{x}} =
\begin{bmatrix}
\frac{\partial J}{\partial x_{1}} \\
\frac{\partial J}{\partial x_{2}}
\end{bmatrix}
=
\begin{bmatrix}
\frac{\partial y_{1}}{\partial x_{1}} & \frac{\partial y_{1}}{\partial x_{2}} \\
\frac{\partial y_{2}}{\partial x_{1}} & \frac{\partial y_{2}}{\partial x_{2}} \\
\frac{\partial y_{3}}{\partial x_{1}} & \frac{\partial y_{3}}{\partial x_{2}} \\
\end{bmatrix} ^{T}
\begin{bmatrix}
\frac{\partial J}{\partial y_{1}} \\
\frac{\partial J}{\partial y_{2}} \\ 
\frac{\partial J}{\partial y_{3}} \\
\end{bmatrix}

&=
W^{T}\frac{\partial J}{\partial \textbf{y}}
\end{align}
$$
  
從以上結果得知，要先將 `weight` 轉置，再和 $$\frac{\partial J}{\partial \textbf{y}}$$ 做 矩陣-向量 乘積。


如果 $$W$$ 和 $$\frac{\partial J}{\partial \textbf{y}}$$ 的值分別如下：

$$
\begin{align}
& \textbf{W} =
\begin{bmatrix}
 0.0635  & 0.4911 \\
-0.1080  & 0.1747 \\
  0.2063 & -0.1635 \\
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
W^{T}\frac{\partial J}{\partial \textbf{y}}
=
\begin{bmatrix}
 0.0635 &-0.1080 & 0.2063 \\
 0.4911 & 0.1747 &-0.1635 \\
\end{bmatrix}
\begin{bmatrix}
1 \\
2 \\
3 \\ 
\end{bmatrix}
=
\begin{bmatrix}
 0.0635 \times 1 &-0.1080 \times 2 & 0.2063 \times 3 \\
 0.4911 \times 1 & 0.1747 \times 2 &-0.1635 \times 3 \\
\end{bmatrix}
=
\begin{bmatrix}
 0.4665 \\
 0.3501 \\
\end{bmatrix}
$$


此部分，程式實作如下：


``` lua

dj_dy = torch.Tensor{1,2,3}
x = torch.Tensor{1,1}
print( l2:updateGradInput( x,dj_dy ))

```


結果如下：


``` sh

 0.4665
 0.3501
[torch.DoubleTensor of size 2]

```

以上程式碼，輸入 `x` 只是為了滿足 `input:dim() == 1` 的條件， 而 `x` 的數值是不會影響結果的。


至於 `input:dim() == 2` 的情況，推導方式同上，在此不做詳細推倒。



再來看 `accGradParameters` 的程式碼：


``` nn/Linear.lua

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
