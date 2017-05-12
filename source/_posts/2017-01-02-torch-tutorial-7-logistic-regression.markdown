---
layout: post
title: "Torch NN Tutorial 7 : Multilayer Perceptron"
date: 2017-01-02 11:17:07 +0800
comments: true
categories: ['Neural Networks','Torch']
published: false 
---


本文以多層的類神經網路為例，


``` lua

input= torch.rand(100,2);     
output= torch.Tensor(100,1);
for i =1,input:size()[1] do
    if (input[i][1] > 0.5 and input[i][2] > 0.5) or
       (input[i][1] < 0.5 and input[i][2] < 0.5)then  
        output[i] = 0
    else
        output[i] = 1
    end
end

```





``` lua

require "nn"
mlp = nn.Sequential();  
inputs = 2; outputs = 1; HUs = 2; 
mlp:add(nn.Linear(inputs, HUs))
mlp:add(nn.Sigmoid())
mlp:add(nn.Linear(HUs, outputs))
mlp:add(nn.Sigmoid())

```





``` lua

for i = 1,2500 do

  criterion:forward(mlp:forward(input), output)
  mlp:zeroGradParameters()
  mlp:backward(input, criterion:backward(mlp.output, output))
  mlp:updateParameters(0.01)
end

```

