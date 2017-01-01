---
layout: post
title: "Torch NN Tutorial 3 : NN.Criterion & NN.MSECriterion"
date: 2016-12-25 18:48:14 +0800
comments: true
categories: ['Neural Networks','Torch']
---

## Introduction

torch 的 `nn.Module` 是組成 neural network 的部分，但是要訓練一個 neural network ，就需要有 loss function 。而 `nn.Criterion` 就是用來計算 loss function 的值。`nn.Criterion` 是個抽象類別，所有種類的 loss function 都繼承於它。

例如， loss funciton 用 Minimum Square Error 時， $$\bar{y}$$ 為模型預測出來的數值， $$y$$ 為正確的數值 ，則 loss function $$ L (y, \bar{y}) $$ 的公式如下：

$$

l( y , \bar{y}) = (y  - \bar{y} )^2 

$$

在 torch 中，負責計算 Minimum Square Error 的 criterion 為 `nn.MSECriterion` 。


<!-- more -->


實作部份如下，首先，載入 `nn` 模組：


``` lua

require 'nn'

```


建立 `nn.MSECriterion` ，命名為 `c1` ，如下：


``` lua

c1 = nn.MSECriterion()

```


假設模型預測出的數值 $$ \bar{y} = 5$$ ，正確數值 $$y=3$$ ，則：


$$

l( y , \bar{y}) = (y  - \bar{y} )^2  = (3-5)^2 = 4

$$ 


實作如下，將 `y_ = 5` 與 `y = 3` 兩數值傳入 `c1` ，進行 forward propagation ，得出 loss function `l` 的數值。


``` lua

y_ = torch.Tensor{5}
y = torch.Tensor{3}
l = c1:forward(y_,y)
print(l)

```


執行結果， `l = 4` ，如下：


``` sh

4	

```


## nn.Criterion & nn.MSECriterion


這邊講解 `nn.Criterion` 以及 `nn.MSECriterion` 的程式碼：

`nn.Criterion` 程式碼：[https://github.com/torch/nn/blob/master/Criterion.lua](https://github.com/torch/nn/blob/master/Criterion.lua)

`nn.MSECriterion` 程式碼： [https://github.com/torch/nn/blob/master/MSECriterion.lua](https://github.com/torch/nn/blob/master/MSECriterion.lua)


首先看到 `nn.Criterion` 的部分，先從 `init` 開始。


``` lua nn/Criterion.lua

local Criterion = torch.class('nn.Criterion')

function Criterion:__init()
   self.gradInput = torch.Tensor()
   self.output = 0
end

```


從以上程式碼得知， `nn.Criterion` 雖然有 `output` 和 `gradInput` 這兩個變量，但它並沒有繼承 `nn.Module` 。


再來看到 forward propagation 的部分， `nn.Criterion` 的 `forward` 和 `nn.Module` 的不一樣，因為它的輸入有 `input` 和 `target` 這兩個變量，如下：

因為 loss function 的 forward propagation 需要有 input 和 target 這兩個數值才算得出來，而 module 中的 forward propagation 部分，通常只需要 input 的數值即可。


``` lua nn/Criterion.lua

function Criterion:forward(input, target)
   return self:updateOutput(input, target)
end

```


再來看到 `nn.MSECriterion` 的程式碼。


``` lua nn/MSECriterion.lua

function MSECriterion:__init(sizeAverage)
   parent.__init(self)
   if sizeAverage ~= nil then
     self.sizeAverage = sizeAverage
   else
     self.sizeAverage = true
   end
end


```

其中， `sizeAverage` 是設定要不要將 output 根據 input 的維度來做平均，預設值為 `true` 。

舉個例子，`y2_` 和 `y2` 分別為 input 與 target ，分別建立 `c2` 與 `c3` 兩個 `nn.MSECriterion` ， `c2` 的 `sizeAverage` 為 `true` ，而 `c3` 的 `sizeAverage` 為 `false` 。


``` lua

y2_ = torch.Tensor{5,5}
y2 = torch.Tensor{3,3}
c2 = nn.MSECriterion(true)
c3 = nn.MSECriterion(false)

```

將 `y2_` 和 `y2` 都輸入 `c2` 和 `c3` ，進行 forward propagation ，比較其結果差異，程式碼如下：


``` lua

print(c2:forward(y2_,y2))
print(c3:forward(y2_,y2))

```


則輸出結果如下：


``` sh

4	
8

```


以上， 4 是 `sizeAverage` 為 `true` 的結果，也就是將 `y2_` 和 `y2` 的每個元素相減後再平均，如下：


$$

\frac{(5-3)^2 + (5-3)^2 }{2} = 4

$$


而 8 是 `sizeAverage` 為 `false` 的結果，也就是將 `y2_` 和 `y2` 的每個元素相減，但沒平均，如下：


$$

(5-3)^2 + (5-3)^2  = 8

$$


這裡要注意的是，不管 `sizeAverage` 是 `ture` 或 `false` ，也不論 input 的維度有多少，output 都是一維的純量。


再來看到 `updateOutput` 的程式碼：


``` lua nn/Criterion.lua

function MSECriterion:updateOutput(input, target)
   self.output_tensor = self.output_tensor or input.new(1)
   input.THNN.MSECriterion_updateOutput(
      input:cdata(),
      target:cdata(),
      self.output_tensor:cdata(),
      self.sizeAverage
   )
   self.output = self.output_tensor[1]
   return self.output
end

``` 


此部分的核心運算是用 C 來加速，在第三行會呼叫 C 的函數 `MSECriterion_updateOutput` ，再將結果存在 `output_tensor` ，而 lua 則從 `output_tensor` 中取出結果，傳到 `output` 。

C 的程式碼在 [MSECriterion.c](https://github.com/torch/nn/blob/master/lib/THNN/generic/MSECriterion.c) 檔案中：


``` c  nn/lib/THNN/generic/MSECriterion.c

void THNN_(MSECriterion_updateOutput)(
          THNNState *state,
          THTensor *input,
          THTensor *target,
          THTensor *output,
          bool sizeAverage)
{
  THNN_CHECK_NELEMENT(input, target);
  THNN_CHECK_DIM_SIZE(output, 1, 0, 1);

  real sum = 0;

  TH_TENSOR_APPLY2(real, input, real, target,
    real z = (*input_data - *target_data);
    sum += z*z;
  );

  if (sizeAverage)
    sum /= THTensor_(nElement)(input);

  THTensor_(set1d)(output, 0, sum);
}

```


在第 14 ~ 15 行中可看到將 `input_data` 與 `target_data` 相減後得出 `z` ，並將 `z` 平方後累加，得到 `sum` ，而在 18 ~ 19 行中， `sum` 會根依序 `sizeAverage` 的值，來決定要不要除以所有元素的個數。

最後一行則將 `sum` 的值存到 `output` 中，這個 `output` 所對應到 lua 中的 `output_tensor` 。


關於 `updateGradInput` 以及 backward propagation 的部分，將在下回介紹。


## Materials

本次教學的完整程式碼於此：

[https://github.com/ckmarkoh/torch_nn_tutorials/blob/master/3_nn_criterion_and_msecriterion.ipynb](https://github.com/ckmarkoh/torch_nn_tutorials/blob/master/3_nn_criterion_and_msecriterion.ipynb)