---
layout: post
title: "Linear Programming 2: Fundamental Theorem of Linear Programming"
date: 2017-05-31 01:33:37 +0800
comments: true
categories: ['Optimization Methods','Linear Programming']
published: false 
---


## Introduction

本文接續前文 [Linear Programming 1: Standard Form of Linear Programming](/blog/2017/05/30/linear-programming) ，介紹Linear Programming 的最佳解有什麼特性。

Fundamental Theorem of Linear Programming 為：Linear Programming 的最佳解，必會出現在 Feasible Domain 的 Extreme Point （頂點）。

前文提到的 Linear Programming 問題，可以把它轉成 Standard Form 。

$$
\begin{align}
&\text{Objective Function} \\
&\mspace{20mu}\text{minimize: } -2x_1-3x_2  \\
&\text{Constraints} \\
&\mspace{20mu} x_1 + x_2 + x_3 =  2 \\
&\mspace{20mu} x_1 + 2x_2 + x_4 = 3 \\
&\mspace{20mu} x_1 , x_2 , x_3 , x_4 \geq 0  \\
\end{align}
$$

轉成 Standard Form 後，就比較方便來討論 Linear Programming 的最佳解有什麼特性。

## Geometry of Linear Programming 

先來看 Feasible Domain 有什麼樣的幾何性質：

首先看平面 $$x_1 + x_2 + x_3 =  2$$  的部分：


$$
x_1 + x_2 + x_3 =  2 
x_1 , x_2 , x_3 geq 0
$$


此為三維空間中的一個平面


Equality Constraint 不會因為


由於此問題共有四個維度：$$x_1,x_2,x_3,x_4$$ ，沒辦法完整把它的圖形化出來，
但我們可以畫出它投影到 $$x_1$$ 和 $$x_2$$ 這兩個維度上的圖，如下：


![](/images/new/linearprog2.png)


其中，淺藍色區域 $$\mathbf{P}$$ 是 $$(x_1,x_2)$$ 滿足 Constraints 的範圍，稱為 Feasible Domain。
Feasible Domain  Convex Set ，也就是說，在



