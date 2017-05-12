---
layout: post
title: "Torch NN Tutorial 5 : Backward Propagation ( part 2 : nn.Criterion ) "
date: 2017-01-01 15:17:57 +0800
comments: true
categories: ['Neural Networks','Torch']
---


## Backward Propagation in nn.Criterion


本文接續 [Torch NN Tutorial 4: Backward Propagation (part 1)](/blog/2017/01/01/torch-nn-tutorial-4-backward-propagation/) ，講解 `nn.Criterion` 中，與 backward propagation 相關的程式碼。

Criterion 的 `forward`  是負責計算 loss funciton $$J$$ 的值，而 `backward` 則是計算 $$\frac{\partial J}{\partial \bar{y}}$$ 的值。其中， $$\bar{y}$$ 為模型預測出的結果。

`nn.Criterion` 程式碼：[https://github.com/torch/nn/blob/master/Criterion.lua](https://github.com/torch/nn/blob/master/Criterion.lua)


`backward` 的部分，如下：


``` lua nn/Criterion.lua

function Criterion:backward(input, target)
   return self:updateGradInput(input, target)
end

function Criterion:updateGradInput(input, target)
end

```


`nn.Criterion` 的 `backward` 只做一件事，也就是 `updateGradInput` ，而  `updateGradInput` 的運算內容則由繼承它的類別來實作。


<!--more-->


## Backward Propagation in nn.MSECriterion

本文舉 `nn.MSECriterion` 為例。

`nn.MSECriterion` 程式碼：[https://github.com/torch/nn/blob/master/MSECriterion.lua](https://github.com/torch/nn/blob/master/MSECriterion.lua)


`updateGradInput` 的部分，如下：


``` lua nn/MSECriterion.lua

function MSECriterion:updateGradInput(input, target)
   input.THNN.MSECriterion_updateGradInput(
      input:cdata(),
      target:cdata(),
      self.gradInput:cdata(),
      self.sizeAverage
   )
   return self.gradInput
end

```


此部分的運算由 C 來實作，運算完後，回傳結果 `gradInput` 。

C 程式碼： [https://github.com/torch/nn/blob/master/lib/THNN/generic/MSECriterion.c](https://github.com/torch/nn/blob/master/lib/THNN/generic/MSECriterion.c)


``` c nn/lib/THNN/generic/MSECriterion.c

void THNN_(MSECriterion_updateGradInput)(
          THNNState *state,
          THTensor *input,
          THTensor *target,
          THTensor *gradInput,
          bool sizeAverage)
{
  THNN_CHECK_NELEMENT(input, target);
  
  real norm = (sizeAverage ? 2./((real)THTensor_(nElement)(input)) : 2.);

  THTensor_(resizeAs)(gradInput, input);
  TH_TENSOR_APPLY3(real, gradInput, real, input, real, target,
    *gradInput_data = norm * (*input_data - *target_data);
  );
}

```


主要的運算是由第13行的 `TH_TENSOR_APPLY3` 來執行，這個 function 的意思是，執行某個 function 在三個 tensor 上。而要執行的 function 是 `*gradInput_data = norm * (*input_data - *target_data)` ，它需要 tensor ，分別是 `input_data` ， `target_data` 和 `gradInput_data` 。

從數學公式上來看，公式如下：


$$
\begin{align}
& \frac{\partial J}{\partial \bar{y} } = \frac{ \partial (\bar{y} -y)^2  }{\partial \bar{y}} \\
& = 2(\bar{y} -y)
\end{align}
$$


其中， $$ \frac{\partial J}{\partial \bar{y} } $$ 為 `gradInput_data` ， $$\bar{y}$$ 為 `input_data` ，而 $$y$$ 為 `target_data` 。  $$2$$ 為常數 `norm` 。


如果 $$ \bar{y} $$ 和 $$y$$ 皆為向量，則 $$ \frac{\partial J}{\partial \bar{y} } $$ 則是針對 $$ \bar{y} $$ 中的每個元素來分別計算，如果 `sizeAverage` 為 true 的話，`norm` 還要除以向量的長度。舉以下例子，$$ \bar{y} = [1,1,1], y=[1,2,3] $$  則 ：


$$
\frac{\partial J}{\partial \bar{y} } = 
\begin{bmatrix}
\frac{\partial J}{\partial \bar{y}_{1} } \\
\frac{\partial J}{\partial \bar{y}_{2} } \\ 
\frac{\partial J}{\partial \bar{y}_{3} } \\
\end{bmatrix} 
=
\frac{2}{3}
\begin{bmatrix}
\bar{y}_{1} - y_{1} \\
\bar{y}_{2} - y_{2} \\
\bar{y}_{3} - y_{3} \\
\end{bmatrix} 
=
\frac{2}{3}
\begin{bmatrix}
1 - 1 \\
1 - 2 \\
1 - 3 \\
\end{bmatrix} 
=
\begin{bmatrix}
0.0000 \\
-0.6667 \\
-1.3333 \\
\end{bmatrix}
$$


其中， $$\bar{y}_{1}$$ 代表 $$\bar{y}$$ 中的第一的元素，而前面乘上 $$\frac{2}{3}$$ 是因為 `sizeAverage` 為 `true` ，所以微分的結果要除以向量長度 3。

以 `backward` 執行以上運算，程式如下：


``` lua

y_ = torch.Tensor{
    {1},{1},{1}
}

y = torch.Tensor{
    {1},{2},{3}
}

c1 = nn.MSECriterion()
c1:backward(y_,y)
print(c1.gradInput)

```


結果如下：


``` sh

 0.0000
-0.6667
-1.3333
[torch.DoubleTensor of size 3x1]

```


如果 `sizeAverage` 為 `false` 的話， 則不需再除以 `input` 的大小， $$ \frac{\partial J}{\partial \bar{y} } $$ 的結果如下：


$$
\frac{\partial J}{\partial \bar{y} } = 
\begin{bmatrix}
\frac{\partial J}{\partial \bar{y}_{1} } \\
\frac{\partial J}{\partial \bar{y}_{2} } \\ 
\frac{\partial J}{\partial \bar{y}_{3} } \\
\end{bmatrix} 
=
2
\begin{bmatrix}
\bar{y}_{1} - y_{1} \\
\bar{y}_{2} - y_{2} \\
\bar{y}_{3} - y_{3} \\
\end{bmatrix} 
=
2
\begin{bmatrix}
1 - 1 \\
1 - 2 \\
1 - 3 \\
\end{bmatrix} 
=
\begin{bmatrix}
0 \\
-2 \\
-4 \\
\end{bmatrix}
$$


將 `sizeAverage` 設為 `false` 的方法，即是在建立 `nn.MSECriterion` 時，輸入 `false` ，方法如下：


``` lua

c2 = nn.MSECriterion(false)
c2:backward(y_,y)
print(c2.gradInput)

```

以 `backward` 執行運算，結果如下：


``` sh

 0
-2
-4
[torch.DoubleTensor of size 3x1]

```

## Materials

本次教學的完整程式碼於此：

[https://github.com/ckmarkoh/torch_nn_tutorials/blob/master/5_backward_propagation_part_2.ipynb
](https://github.com/ckmarkoh/torch_nn_tutorials/blob/master/5_backward_propagation_part_2.ipynb
)


