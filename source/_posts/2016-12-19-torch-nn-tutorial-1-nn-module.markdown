---
layout: post
title: "Torch NN tutorial 1 : NN.Module & NN.Linear"
date: 2016-12-19 22:36:47 +0800
comments: true
categories: ['Neural Networks','Torch']
---

## Introduction

此系列講解如何用 torch 實作 neural network 。

本系列不講解如何安裝 torch 及 lua 的基本語法，假設讀者都已具備這些基礎知識。

以 torch 實作 neural network 時，最常用的套件為 [nn](https://github.com/torch/nn)，而在 `nn` 中，建構 neural network 最基本的單位為 [nn.Module](https://github.com/torch/nn/blob/master/Module.lua) 。 `nn.Module` 是一個抽象類別，所有建構 neural network 本身有關的 module ，都是從 `nn.Module` 所繼承而來。


舉個例子，如果要實作以下運算：

$$

\textbf{y} = \textbf{W}\textbf{x} + \textbf{b} 

$$

假設 $$\textbf{x}$$ 為 2 維的 input ，而 $$\textbf{y}$$ 為 3 維的output， $$\textbf{W},\textbf{b}$$ 分別為 weight 和 bias ，此兩參數皆以隨機值進行初始化。


使用 torch 實作此運算的方法如下：

首先，載入 nn 套件：


```lua

require 'nn'

```


建立一個 Linear Module：


```lua

l1 = nn.Linear(2,3)

```


<!--more-->


其中， [nn.Linear](https://github.com/torch/nn/blob/master/Linear.lua) 即是用來進行 $$\textbf{y} = \textbf{W}\textbf{x} + \textbf{b} $$ 這類的線性運算所用的模組，它繼承了 `nn.Module` 。 而 2 和 3 分別代表了 $$\textbf{x}$$ 和 $$\textbf{y}$$ 的維度。 當它被建構出來時， weight 和 bias 的值會以隨機值來初始化。 

以上程式中，建立一個命名為 `l1` 的 module ，如果要取得它的 weight 和 bias ，可以用 `l1.weight` 和 `l1.bias` 取得，方法如下：


```lua

print(l1.weight)
print(l1.bias)

```


執行結果如下：


```sh

 0.1453  0.5062
 0.0635  0.4911
-0.1080  0.1747
[torch.DoubleTensor of size 3x2]

 0.2063
-0.1635
-0.0883
[torch.DoubleTensor of size 3]

```


其中， size 3x2 的 tensor 為 weight, size 3 的 tensor 為 bias。

用此 module 可以執行運算，令 x 為一個二維向量 $$[0,1]$$ ，輸入此 module ，進行 forward propagation ，也就是說，執行 $$\textbf{y} = \textbf{W}\textbf{x} + \textbf{b} $$ 的運算， 並輸出結果為 $$\textbf{y}$$  ，實作如下：


```lua

x = torch.Tensor{0,1}
y = l1:forward(x)
print(y)

```


輸出結果 `y` 為一個三維向量，如下：


```sh

 0.7125
 0.3276
 0.0865
[torch.DoubleTensor of size 3]

```


## nn.Module & nn.Linear

這邊要更進一步介紹 `nn.Module` 和 `nn.Linear` 的內容是什麼。由於 torch 的程式碼相當簡潔易懂，可以直接看程式碼來了解它的功能是什麼。

`nn.Module` 程式碼： [https://github.com/torch/nn/blob/master/Module.lua](https://github.com/torch/nn/blob/master/Module.lua)

`nn.Linear` 程式碼： [https://github.com/torch/nn/blob/master/Linear.lua](https://github.com/torch/nn/blob/master/Linear.lua)

首先，介紹 `nn.Module` ，先看 `init()` 的部分：


```lua nn/Module.lua

function Module:__init()
   self.gradInput = torch.Tensor()
   self.output = torch.Tensor()
   self._type = self.output:type()
end

```


`Module` 中最基本的成員有 `output` 和 `gradInput` 。
`output` 為此 `Module` 的 forward propagation 結果，而 `gradInput` 為 backward propagation 的運算結果。
這些變量一開始都會被初始化為 空的 tensor 。

註：本文先不講解 backward propagation 與 `gradInput` 的部分，交由之後的教學文章來解釋。


在 `Module:forward` 的部分，是用來進行 forward propagation的，如下：


```lua nn/Module.lua

function Module:updateOutput(input)
   return self.output
end

function Module:forward(input)
   return self:updateOutput(input)
end

```


先看 forward 的部分， Module 沒有運算的實作，僅單純輸出 `output` 值。如果呼叫了 forward propagation ，則從 `Module:updateOutput` 就直接輸出了 `output` 。

而 `nn.Linear` 則實作了 forward propagation。

所謂的 Linear，即是指 $$\textbf{y} = \textbf{W}\textbf{x} + \textbf{b} $$ 的線性運算。

再來看 `nn.Linear` 的程式碼，先看 `init()` 的部分：


```lua nn/Linear.lua

local Linear, parent = torch.class('nn.Linear', 'nn.Module')

function Linear:__init(inputSize, outputSize, bias)
   parent.__init(self)
   local bias = ((bias == nil) and true) or bias
   self.weight = torch.Tensor(outputSize, inputSize)
   self.gradWeight = torch.Tensor(outputSize, inputSize)
   if bias then
      self.bias = torch.Tensor(outputSize)
      self.gradBias = torch.Tensor(outputSize)
   end
   self:reset()
end

```


在第1行， `nn.Linear` 繼承了 `nn.Module` 。

在第3行開始可以看到，建構 Linear 所需的參數有 `inputSize` , `outputSize` 和 `bias` 。 `bias`  不一定要給，如果沒有給，則預設值會讓它是隨機的。除非 `bias=false` ，則此 Linear Module 就不會有 `bias` 。
從6~10行中，它比 `nn.Module` 多了 `weigt` 和 `bias` 這兩個變量，而 `reset()` 則是將它們初始化。

如果要建立一個 Linear Module，則要給定 `inputSize` 和 `outputSize` ，也就是 $$\textbf{x}$$ 和 $$\textbf{y}$$ 的維度。


假設  $$\textbf{x}$$ 是二維， $$\textbf{y}$$ 是三維，建立一個命名為 `l2` 的 Linear 模組：


```lua

l2 = nn.Linear(2,3)

```



用以下方法印出 l2 的 `weight` , `bias` 和 `output` ：


```lua

print(l2.weight)
print(l2.bias)
print(l2.output)

```

輸出結果如下：


```sh

-0.2863  0.5541
-0.6269  0.6557
-0.3215 -0.1648
[torch.DoubleTensor of size 3x2]

-0.0316
 0.4126
 0.4415
[torch.DoubleTensor of size 3]

[torch.DoubleTensor with no dimension]

```


其中，`weight` 和 `bias` 會被初始化隨機成 size 3x2 和 size 3 的 double tensor ，而最後一行顯示出 `output` 還是空的（with no dimension）。


如果想知道各個 variable 的 size ，還有個方式，就是直接用 print 的方式把它印出來，作法如下：


```lua

print(l2)

```


執行結果如下：


```sh

nn.Linear(2 -> 3)
{
  gradBias : DoubleTensor - size: 3
  weight : DoubleTensor - size: 3x2
  _type : torch.DoubleTensor
  output : DoubleTensor - empty
  gradInput : DoubleTensor - empty
  bias : DoubleTensor - size: 3
  gradWeight : DoubleTensor - size: 3x2
}

```


要讓 `output` 有值，就要進行 forward propagation 。而 `Linear:updateOutput` 則是實作了 `Module:updateOutput` 中， forward propagation 運算的實際內容，程式碼如下：


```lua nn/Linear.lua

function Linear:updateOutput(input)
   if input:dim() == 1 then
      self.output:resize(self.weight:size(1))
      if self.bias then self.output:copy(self.bias) else self.output:zero() end
      self.output:addmv(1, self.weight, input)
   elseif input:dim() == 2 then
      local nframe = input:size(1)
      local nElement = self.output:nElement()
      self.output:resize(nframe, self.weight:size(1))
      if self.output:nElement() ~= nElement then
         self.output:zero()
      end
      updateAddBuffer(self, input)
      self.output:addmm(0, self.output, 1, input, self.weight:t())
      if self.bias then self.output:addr(1, self.addBuffer, self.bias) end
   else
      error('input must be vector or matrix')
   end

   return self.output
end

```


以上可以分為兩部分來看，首先是當 `input:dim() ==1` 時，也就是 `input` 的維度為 1 ，也就是一次只輸入單筆資料的時候。第一步，會先調整 `output` 的 `size` 為適當的大小，再將 `bias` 複製到 `output` ，再讓 `input` 和 `weight` 進行矩陣相乘，並和 `output` 相加，再輸出結果。

從數學公式上來看，輸入值 $$\textbf{x}$$ 是一維的向量，進行的運算如下：

$$
\textbf{W}\textbf{x} + \textbf{b}
$$

此時， $$\textbf{W}$$ 放前面，而 $$\textbf{x}$$ 放後面。

例如當輸入值向量 $$[0,1]$$ 時，則矩陣運算的結果為：

$$
\begin{align}
&\begin{bmatrix}
-0.2863 & 0.5541 \\
-0.6269 & 0.6557 \\
-0.3215 & -0.1648 \\
\end{bmatrix}
\begin{bmatrix}
0 \\
1
\end{bmatrix}
+ 
\begin{bmatrix}
-0.0316 \\
 0.4126 \\
 0.4415 \\
\end{bmatrix}
=
\begin{bmatrix}
-0.2863 \times 0 + 0.5541 \times 1  -0.0316 \\
-0.6269 \times 0 + 0.6557 \times 1 + 0.4126\\
-0.3215 \times 0  -0.1648 \times 1 + 0.4415\\
\end{bmatrix}\\
&=
\begin{bmatrix}
0.5541 -0.0316 \\
0.6557 + 0.4126\\
-0.1648 + 0.4415\\
\end{bmatrix}
=
\begin{bmatrix}
  0.5225 \\
  1.0683  \\
  0.2766 \\
\end{bmatrix}
\end{align}

$$

實作以上算式，呼叫 `l2:forward` ，輸入 `torch.Tensor{0,1}` 並印出結果：


```lua

print(l2:forward(torch.Tensor{0,1}))

```


結果如下：


```sh

 0.5225
 1.0683
 0.2766
[torch.DoubleTensor of size 3]

```


這時已經進行過了 forward 運算，而 `output` 有值了，所以可以印出它的值，方法如下：


```lua

print(l2.output)

```


`output` 的值也是如下：


```sh

 0.5225
 1.0683
 0.2766
[torch.DoubleTensor of size 3]

```


而當 `input:dim() ==2` 時，也就是 `input` 的維度為 2 ，也就是一次輸入多筆資料。這時需要先把 `weight` 進行轉置，再和 `input` ，並和 `bias` 相加，並將結果加到 `output` 並輸出。

一次輸入多筆資料的目的是為了加速運算，因為用矩陣對矩陣的相乘的方式就可以來加速。這樣同時輸入的一批資料，就稱為 batch 。

從公式上來看，輸入值 $$\textbf{X}$$ 是二維的矩陣，則進行以下矩陣運算：

$$
\textbf{X}\textbf{W}^{T} + \textbf{B}
$$

此時， $$\textbf{X}$$ 放前面，而 $$\textbf{W}$$ 進行轉置後放後面。

而 $$\textbf{B}$$ 是將 $$\textbf{b}$$ 轉置以後，再複製其橫排所形成的矩陣，以便和前面的矩陣相乘結果來相加。

例如當輸入資料有兩筆向量 $$[0, 1]$$ 和 $$[2, 1]$$ 時，則可以組成以下矩陣 (2x2) ：


$$
\begin{bmatrix}
0 &1 \\
2 &1 \\
\end{bmatrix}


$$ 

則矩陣運算的過程如下：

$$
\begin{align}
&
\begin{bmatrix}
0 &1 \\
2 &1 \\
\end{bmatrix}
\begin{bmatrix}
-0.2863 & -0.6269 & -0.3215 \\
 0.5541 & 0.6557 & -0.1648 \\
 \end{bmatrix}
+ 
\begin{bmatrix}
-0.0316 & 0.4126 & 0.4415 \\
-0.0316 & 0.4126 & 0.4415 
\end{bmatrix}\\
&
=
\begin{bmatrix}
   -0.2863 \times 0 +  0.5541 \times 1
&  -0.6269 \times 0 +  0.6557 \times 1
&  -0.3215 \times 0   -0.1648 \times 1\\

   -0.2863 \times 2 +  0.5541 \times 1
&  -0.6269 \times 2 +  0.6557 \times 1
&  -0.3215 \times 2   -0.1648 \times 1\\
\end{bmatrix}\\
&
+ 
\begin{bmatrix}
-0.0316 & 0.4126 & 0.4415 \\
-0.0316 & 0.4126 & 0.4415 
\end{bmatrix}\\
&
=
\begin{bmatrix}
0.5541 -0.0316
& 0.6557 +0.4126
& -0.1648 +0.4415 \\
-0.0186  -0.0316
& -0.5981 +0.4126
& -0.8079 +0.4415 \\
\end{bmatrix}\\
&
=
\begin{bmatrix}
 0.5225 & 1.0683  & 0.2766 \\
-0.0502 & -0.1855 & -0.3664 \\
\end{bmatrix}
\end{align}\\

$$

輸出結果為一個 2x3 的矩陣，每一個橫排為一個三維向量，代表著每一筆資料經過線性運算的結果。


實作以上算式，呼叫 `l2:forward` ，輸入由 `{0,1}` 和 `{2,1}` 這兩筆資料組成的 batch， 並印出結果：


{% raw %}
```lua

input={
   {0,1},
   {2,1}
}
print(l2:forward(torch.Tensor(input)))

```
{% endraw %}


結果如下：


```sh

 0.5225  1.0683  0.2766
-0.0502 -0.1855 -0.3664
[torch.DoubleTensor of size 2x3]

```


也可印出 `output` 的值：


```lua

print(l2.output)

```


結果如下：


```sh

 0.5225  1.0683  0.2766
-0.0502 -0.1855 -0.3664
[torch.DoubleTensor of size 2x3]

```





## Materials

本次教學的完整程式碼於此：

[https://github.com/ckmarkoh/torch_nn_tutorials/blob/master/1_nn_module_and_linear.ipynb](https://github.com/ckmarkoh/torch_nn_tutorials/blob/master/1_nn_module_and_linear.ipynb)



