---
layout: post
title: "Torch NN Tutorial 2 : NN.Container & NN.Sequential"
date: 2016-12-24 00:24:25 +0800
comments: true
categories: ['Neural Networks','Torch']
---

## Introduction

在[Torch NN tutorial 1 : NN.Module & NN.Linear](/blog/2016/12/19/torch-nn-tutorial-1-nn-module/)中，介紹了構成 neural network 的最基本的單位，也就是 `nn.Module` ，並介紹了 `nn.Linear` 可以進行一次線性運算，但通常一個 neural network 是經由多個線性和非線性的運算構成的。要把多個 Module 串接起來，就需要用到 `nn.Contaner` 。

例如，有個 neural network 的輸入為 x ，輸出為 z ，中間經過了一次線性運算與一個sigmoid function ，如下：


$$
\begin{align}
&\textbf{y} = \textbf{W}\textbf{x} + \textbf{b}  \\
&\textbf{z} = \frac{1}{1+e^{-\textbf{y}}}
\end{align}
$$


假設 $$\textbf{x}$$ 為 2 維的向量，而 $$\textbf{y}$$ 為 3 維向量， 
$$\textbf{W},\textbf{b}$$ 分別為 weight 和 bias ，此兩參數皆以隨機值進行初始化，
$$\textbf{z}$$  是 $$\textbf{y}$$ 經過 sigmoid 非線性運算的輸出結果。

<!--more-->

使用 torch 實作此運算：

首先，載入 `nn` 套件：


``` lua

require 'nn'

```


建立以上兩個運算所需的Module，分別為 `nn.Linear` 和 `nn.Sigmoid` ，並將它們分別命名為 `l1` 和 `s1` ，如下：


``` lua

l1 = nn.Linear(2,3)
s1 = nn.Sigmoid()

```

其中， `nn.Sigmoid` 為進行 sigmoid 運算所需的 module 。


如果要進行運算，方法如下：

假設 x 為一個二維向量 $$[0,1]$$ ，先輸入 `l1` ，進行 forward propagation ，將運算結果傳給 `y` ，如下：


``` lua

x = torch.Tensor{0,1}
y = l1:forward(x)
print(y)

```


執行完後會印出 `l1` 的輸出值，也就是 `y` 的值，是個三維的向量，
y 的值會與 `l1.weight` 和 `l1.bias` 的初始值值關， `y` 值如下：


``` sh

 0.1948
 0.9780
 0.3982
[torch.DoubleTensor of size 3]

```


再來將 `y` 輸入到 `s1` 中， 進行 forward propagation，將運算結果傳給 `z` ，如下：


``` lua

z = s1:forward(y)
print(z)

```


執行完後會印出 `z` 值，此值是由 `y` 的值經過 sigmoid function 的運算所得出來的， `z` 值如下：


``` sh

 0.5485
 0.7267
 0.5983
[torch.DoubleTensor of size 3]

```


以上過程中，給定了 `x` 值，經過每一個 module 都要呼叫一次 `forward` ，最後才能取得 `z` 的值，
如果一個 network 是由很多個 module 所組成的，這樣要呼叫很多次 `forward` ，會很麻煩。

而 `nn.Container` 則是一個容器，它可以把多個 module 放進去，並自動處理這些 module 輸入與輸出值的傳遞。 `nn.Container` 也是個抽象個類別，泛指能夠把 module 放進去的容器。

其中一種 container 可將前一個 module 的輸出，傳給下一個 module 的輸入，這種 contanier 為 `nn.Sequential()` 。

例如，如果要將 `l1` 的輸出，傳入 `s1` ，則可以用 `nn.Sequential` 來達成。建立一個命名為 `n1` 的 sequential container ，將 `l1` 和 `s1` 串起來，方法如下：



``` lua

n1 = nn.Sequential()
n1:add(l1)
n1:add(s1)

```



其中，`add` 是將 module 加入 container 的 function。

其實， `nn.Container` 也是從 `nn.Module` 繼承而來的，所以它也有 `forward` ，可以進行 forward propagation ，只要呼叫了 `nn.Container` 的 `forward` ，它就會自動將裡面的 module 進行 forward ，並輸出結果。

如此給定 `x` 之後，只要進行一次 forward 即可取得 `z` 的結果，如下：



``` lua

x = torch.Tensor{0,1}
z = n1:forward(x)
print(z)

```



執行完後，會印出 `z` 的結果，如下：



``` sh

 0.5485
 0.7267
 0.5983
[torch.DoubleTensor of size 3]

```



## nn.Container & nn.Sequential

這邊要介紹 `nn.Container` 和 `nn.Sequential` 的程式碼。

`nn.Container` 程式碼：[https://github.com/torch/nn/blob/master/Container.lua](https://github.com/torch/nn/blob/master/Container.lua)

`nn.Sequential` 程式碼： [https://github.com/torch/nn/blob/master/Sequential.lua](https://github.com/torch/nn/blob/master/Sequential.lua)


首先，介紹 `nn.Container` ，先看 `init()` 的部分：


``` lua nn/Container.lua

local Container, parent = torch.class('nn.Container', 'nn.Module')

function Container:__init(...)
    parent.__init(self, ...)
    self.modules = {}
end

```


第1行處，看到 `nn.Container` 是從 `nn.Module` 繼承而來的，因此它具有 `nn.Module` 的 variable 及 function 。

第5行中，container 的變量 `modules` ，它是一個 lua table ，是用來存放加到 container 中的  module 。

至於，要如何將 module 加到 container 中？可以用 `add` 加入。 `add` 的程式碼如下： 


``` lua nn/Container.lua

function Container:add(module)
    table.insert(self.modules, module)
    return self
end

```


在第2行可看到， `add` 可以把新的 module 加到 `modules` 中。


``` lua nn/Container.lua

function Container:get(index)
    return self.modules[index]
end

```


除了加進去之外，也可以用 `get` 取得 `modules` 中的某個元素，或是用 `size` 取得 `modules` 的大小，程式碼如下：


``` lua nn/Container.lua

function Container:get(index)
    return self.modules[index]
end

function Container:size()
    return #self.modules
end

```


再來是實作的部分，首先我們建立一個 sequential 的 container ，命名為 `n2` ，並以 `size` 取得它所含的 module 個數，如下：


``` lua

n2 = nn.Sequential()
print(n2:size())

```


剛建立時， 由於 `n2` 的 `modules` 是空的，所以 `size` 是 0。執行結果如下：


``` sh

0	

```


再來將 `l1` 和 `s1` 依序加入，再看看 `size` 的變化：


``` lua

n2:add(l1)
n2:add(s1)
print(n2:size())

```


此時已經加入了兩個 module 進去了，所以 `size` 會是 2 ，執行結果如下：


``` sh

2	

```


可以用 `get` 取得加進去的 module ，例如，想取得第一個加進去的，作法如下：


``` lua

print(n2:get(1))

```


第一個加進去的為 `nn.Linear` ，執行結果如下：


``` sh

nn.Linear(2 -> 3)
{
  gradBias : DoubleTensor - size: 3
  weight : DoubleTensor - size: 3x2
  _type : torch.DoubleTensor
  output : DoubleTensor - size: 3
  gradInput : DoubleTensor - empty
  bias : DoubleTensor - size: 3
  gradWeight : DoubleTensor - size: 3x2
}

```


再來看到 `nn.Sequential()` 的程式碼。


``` lua nn/Sequential.lua

local Sequential, _ = torch.class('nn.Sequential', 'nn.Container')

```


第一行顯示 `nn.Sequential` 繼承了 `nn.Container` 。它的 `init` 也與 `nn.Container` 共用，沒有再另外實作。


再來看 `add` 的程式碼。


``` lua nn/Sequential.lua

function Sequential:add(module)
   if #self.modules == 0 then
      self.gradInput = module.gradInput
   end
   table.insert(self.modules, module)
   self.output = module.output
   return self
end

```


在 `nn.Sequential` 中， `add` 除了會將 module 加到 `modules` 之外，它的 `output` 會是最後一個加進去的 module 的 `output` 。

以下實作，將 `n2` 的 `output` 和它裡面的兩個 module 的 `output` 都印出來看看：


``` lua

print(n2:get(1).output)
print(n2:get(2).output)
print(n2.output)

```


結果如下， `n2` 的 output 是第二個 module 的 output ：


```sh

 0.1948
 0.9780
 0.3982
[torch.DoubleTensor of size 3]

 0.5485
 0.7267
 0.5983
[torch.DoubleTensor of size 3]

 0.5485
 0.7267
 0.5983
[torch.DoubleTensor of size 3]

```


再來看到 `updateOutput` 的部分：


``` lua nn/Sequential.lua

function Sequential:updateOutput(input)
   local currentOutput = input
   for i=1,#self.modules do
      currentOutput = self:rethrowErrors(self.modules[i], i, 'updateOutput', currentOutput)
   end
   self.output = currentOutput
   return currentOutput
end

```

以上可知，在第3行的 for 迴圈中， `modules` 中的 `module` 會依序進行  `updateOutput` ，並將其 `output` 傳遞到下個 `module` 的 `input` ，而 `nn.Sequential` 的 `output` 會是最後一個 `module` 的 `output`


假設輸入 `n2` 的 `x` 為 $$[1,0]$$ ，則進行 forward propagation ，將輸出結果存到 `z` ，實作如下 ：


``` lua

x = torch.Tensor{1,0}
z = n2:forward(x)
print(z)

```


透過 updateOutput 中的 for 迴圈，它會依序跑完內部所有的 module，並得出最後結果，結果如下：


``` sh

 0.5331
 0.6890
 0.5093
[torch.DoubleTensor of size 3]

```


如果想看 `n2` 中有哪些 module ，以及它們的順序，也可以直接用 `print` 的方式，作法如下：


``` lua 

print(n2)

```


結果如下：


{% codeblock lang:sh %}

nn.Sequential {
  [input -> (1) -> (2) -> output]
  (1): nn.Linear(2 -> 3)
  (2): nn.Sigmoid
}
{
  gradInput : DoubleTensor - empty
  modules : 
    {
      1 : 
        nn.Linear(2 -> 3)
        {
          gradBias : DoubleTensor - size: 3
          weight : DoubleTensor - size: 3x2
          _type : torch.DoubleTensor
          output : DoubleTensor - size: 3
          gradInput : DoubleTensor - empty
          bias : DoubleTensor - size: 3
          gradWeight : DoubleTensor - size: 3x2
        }
      2 : 
        nn.Sigmoid
        {
          gradInput : DoubleTensor - empty
          _type : torch.DoubleTensor
          output : DoubleTensor - size: 3
        }
    }
  _type : torch.DoubleTensor
  output : DoubleTensor - size: 3
}

{% endcodeblock %}


以上結果可分成兩部分來看，第一部分是 `nn.Sequential` 裡面每個 module 的順序，從 input 到 output 之間，依次進行了哪些運算。

第二部份是詳細印出 `nn.Sequential` 中，有哪些成員 ，它有從 `nn.Module` 繼承而來的 `gradInput` 即 `output` ，也有從 `nn.Container` 繼承而來的 `modules` 。而 `modules` 的部分會詳細列出裡面每個 module 中有哪些 variable。

這部分的程式碼於 `tostring` 函式中，程式碼如下：


``` lua nn/Sequential.lua

function Sequential:__tostring__()
   local tab = '  '
   local line = '\n'
   local next = ' -> '
   local str = 'nn.Sequential'
   str = str .. ' {' .. line .. tab .. '[input'
   for i=1,#self.modules do
      str = str .. next .. '(' .. i .. ')'
   end
   str = str .. next .. 'output]'
   for i=1,#self.modules do
      str = str .. line .. tab .. '(' .. i .. '): ' .. tostring(self.modules[i]):gsub(line, line .. tab)
   end
   str = str .. line .. '}'
   return str
end

```


## Materials

本次教學的完整程式碼於此：

[https://github.com/ckmarkoh/torch_nn_tutorials/blob/master/2_nn_container_and_sequential.ipynb](https://github.com/ckmarkoh/torch_nn_tutorials/blob/master/2_nn_container_and_sequential.ipynb)
