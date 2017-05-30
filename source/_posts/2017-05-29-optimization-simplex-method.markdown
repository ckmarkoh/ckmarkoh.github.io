---
layout: post
title: "Simplex Method"
date: 2017-05-29 20:36:05 +0800
comments: true
categories: ['Optimization Methods','Linear Programming']
published: false 
---


## Fundamental Theorem of Linear Programming

根據 Fundamental Theorem of Linear Programming，Linear Programming的最佳解
（如果是有限的話）
必定會落在Feasible Domain的頂點上。也就是說，只要檢查頂點是否為最佳解即可。

例如，以下的Linear Programming問題：

$$

\begin{align}

&\text{minimize: } -2x_1-3x_2  \\
&\text{subject to: }\left\{
\begin{array}{c}
&x_1 + x_2 + x_3 = 2 \\
&x_1 + 2x_2 + x_4 = 3 \\
&x_1 , x_2 , x_3, x_4 \geq 0  \\
\end{array}
\right.
\end{align}

$$

把以上問題的 Feasible Domain 的範圍畫在平面上，得出下圖：

![](/images/new/simplex1.png)

其中，淺藍色區域為 Feasible Domain，而這個區域總共有 $$(0,0,2,3)$$, $$(0,1.5,0.5,0)$$, $$(1,1,0,0)$$, $$(2,0,0,1)$$ 四個頂點。
如果要找出最佳解，可以把以上四個頂點分別代入 $$-2x_1-3x_2$$ ，得出 $$(x_1,x_2,x_3,x_4) = (1,1,0,0)$$ 是最佳解。


但如果頂點很多，就無法用這種方法，這時就需要使用 Simplex Method。

## Simplex Method

Simplex Method 是一種 Linear Programming 的方法。
Simplex Method 的精神在於，不需要把所有的 Feasible Domain 的頂點都挑出來試。
然後在目前的頂點所相鄰的頂點中，去找下一個更好的頂點，
這樣一個步一步找下去，每次都找到更好的頂點，最後就會找到最佳解了。


### Basic Feasible Solution

首先，討論如何求出 Feasible Domain 的頂點。
此頂點即所謂的 Basic Feasible Solution。

什麼樣的點才能成為頂點？
如前例


註：如果為本文不討論 Basic Variable 為 0 的情況。






