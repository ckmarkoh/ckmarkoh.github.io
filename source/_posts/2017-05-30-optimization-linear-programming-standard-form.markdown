---
layout: post
title: "Linear Programming 1: Standard Form of Linear Programming"
date: 2017-05-30 09:05:30 +0800
comments: true
categories: ['Optimization Methods','Linear Programming']
published: false 
---

## What is Linear Programming?

Linear Programming 是一種最佳化問題，在此最佳化問題中。Objective Function 和Constraints 都是線性的。

舉個例子，以下為一個 Linear Programming 的問題<a name="eq1">（公式一）</a>：

$$

\begin{align}
&\text{Objective Function} \\
&\mspace{20mu}\text{maximize: }  2x_1+3x_2  \\
&\text{Constraints} \\
&\mspace{20mu} x_1 + x_2  \leq 2 \\
&\mspace{20mu} x_1 + 2x_2 \leq 3 \\
&\mspace{20mu} x_1 , x_2  \geq 0  \\
\end{align}

$$

將 Constraints 畫在平面上，得出下圖：

![](/images/new/linearprog1.png)

<!--more-->

其中，淺藍色區域 $$\mathbf{P}$$ 是 $$(x_1,x_2)$$ 滿足 Constraints 的範圍，稱為 Feasible Domain。

$$

P = \{(x_1,x_2) \mid x_1+x_2 \leq 2 , x_1+2x_2 \leq 3, x_1,x_2 \geq 0\}

$$

而從 Feasible Domain 中，可以使 Objective Function 得到最大值的 $$(x_1,x_2)$$ 為最佳解。 
此問題的最佳解為 $$(x_1,x_2)=(1,1)$$ ，因為它使 Objective Function 得最大值：

$$
2x_1+3x_2 = 2\times 1 + 3\times 1 = 5
$$

Linear Programming 的解法主要有兩種：分別是Simplex Method 和 Interior Point Method。本文先不講解這兩種方法，而會在後續文章講解。


## Formulate A Linear Programming Problem

在現實生活的應用中，有許多問題是可以用 Linear Programming 來解，本段舉例，如何將有限的資源下將利潤最大化的問題，寫成 Linear Programming 的形式，例如：

某工廠生產兩種產品X和Y ，這兩種產品都需要用到兩種原料 A和B，生產1公噸X需要1公噸的A和1公噸的B，生產1公噸Y需要1公噸的A和2公噸的B。工廠存有原料A有2公噸，B有3公噸。而1公噸X可賣2百萬元，1公噸Y可賣3百萬元。 以工廠的現有的資源，要生產多少X和Y怎樣才能獲得最大利潤?

設工廠要生產 $$x_1$$ 公噸的 X ， $$y_1$$ 公噸的 Y ，才能獲得最大利潤。

目標是要將利潤最大化，由於而1公噸X可賣2百萬元，1公噸Y可賣3百萬元，所以 Objective Function 如下（省略百萬元）：

$$
\begin{align}
\text{maximize: }  2x_1+3x_2  \\
\end{align}
$$

但是 X 和 Y 的產量受限於目前工廠僅存的原料量， $$x_1 + x_2$$ 為原料A的用量，不可多餘2公噸，以此類推，$$x_1 + 2x_2$$ 為原料 B的用量，不可多餘3公噸。又由於產量不可能為負，因此 $$x_1,x_2$$ 都不能為負， 所以 Constraints 如下：

$$
\begin{align}
&\mspace{40mu} x_1 + x_2  \leq 2 \\
&\mspace{40mu} x_1 + 2x_2 \leq 3 \\
&\mspace{40mu} x_1 , x_2  \geq 0  \\
\end{align}
$$

結合以上的 Objective Function 和 Constraints，即為<a href="#eq1">（公式一）</a>的 Linear Programming 問題。

## Standard Form of Linear Programming

在解 Linear Programming 的問題時，會先把問題轉成 Standard Form，使得不同的 Linear Programming 問題能有一致性，以便之後能用相同的演算法來求解。
Linear Programming 的 Standard Form 如下<a name="eq2">（公式二）</a>：


$$

\begin{align}
&\text{1.Objective Function} \\
&\mspace{20mu}\text{minimize: } z = c_1x_1+c_2x_2+...+c_nx_n \\
&\text{2.Equality Constraints} \\
&\mspace{20mu} a_{11}x_1 + a_{12}x_2 +...+a_{1n}x_n = b_1 \\
&\mspace{20mu} a_{21}x_1 + a_{22}x_2 +...+a_{2n}x_n = b_2 \\
&\mspace{20mu} \vdots \\
&\mspace{20mu} a_{m1}x_1 + a_{m2}x_2 +...+a_{mn}x_n = b_m \\
&\text{3.Non-Negative Variables} \\
&\mspace{20mu} x_1 , x_2,...,x_n  \geq 0  \\
\end{align}

$$

其中，$n$ 為 Variables 的個數， $m$ 為 Constraints 的個數。通常 $$n >  m$$ ，這樣才需要從 Feasible Domain 中挑選出最佳解，不然 Feasible Domain 可能會只有一個點 ， 或者根本無解。

而 Standard Form 須滿足以下三個條件：

1. 目標是將 Objective Function 最小化，而非最大化。
2. Constraints 皆為 Equality Constraints ，沒有 Inequality Constraints。
3. Variables 都必須是非負實數。

為了以矩陣運算來加速，也可以把<a href="#eq2">（公式二）</a> 的 Standard Form 寫成矩陣的形式<a name="eq3">（公式三）</a>：


$$
\begin{align}
&\mathbf{c} = \begin{bmatrix}
c_1 \\
c_2 \\
\vdots \\
c_n \\
\end{bmatrix}
\mathbf{x} = \begin{bmatrix}
x_1 \\
x_2 \\
\vdots \\
x_n \\
\end{bmatrix}
\mathbf{b} = \begin{bmatrix}
b_1 \\
b_2 \\
\vdots \\
b_m \\
\end{bmatrix} \\
&\mathbf{A} = \begin{bmatrix}
a_{11} &a_{12} &\cdots &a_{1n} \\
a_{21} &a_{22} &\cdots &a_{2n} \\
\vdots &\vdots &\vdots &\vdots \\
a_{m1} &a_{m2} &\cdots &a_{mn} \\
\end{bmatrix} \\
\end{align}
$$


$$
\begin{align}
&\text{Objective Function} \\
&\mspace{40mu} \text{minimize: } z =\mathbf{c}^T\mathbf{x}  \\
&\text{Equality Constraints} \\
&\mspace{40mu} \mathbf{A}\mathbf{x} = \mathbf{b} \\
&\text{Non-Negative Variables} \\
&\mspace{40mu} \mathbf{x}\geq 0 \\
\end{align}
$$



## Converting into Standard Form


我們可以把前例<a href="#eq1">（公式一）</a>轉換成<a href="#eq2">（公式二）</a> 的 standard form 。


首先，為了滿足條件 1.， 「目標是將 Objective Function 最小化。」
必須將 Objective Function 改為 Minimize。這步驟很簡單，把原本的 Objective Function 乘上負號就可以了：


$$
\begin{align}
&\text{maximize: }  2x_1+3x_2  \\
&\Rightarrow \text{minimize: }  -2x_1-3x_2  \\
\end{align}
$$



下一步，為了滿足條件 2.，「 Constraints 皆為 Equality Constraints 」，則須將 Inequality Constraints 轉換成 Equality Constraints 。 這可以藉由加上 Slack Variable 來達成。

例如 Inequalitn Constraint：

$$
x_1 + x_2  \leq 2 
$$

可以將它加上 Slack Variable ，$$x_3$$ 。它可以補足原本不等式少於2的部分，使之等於2。如下：

$$
\Rightarrow  x_1 + x_2 + x_3  = 2 , \mspace{20mu} x_3 \geq 0 
$$


同理，另一個 Constraints 也可以加上另一個 Slack Variable ，$$x_4$$ 。


$$
\begin{align}
&x_1 + 2x_2 \leq 3 \\
&\Rightarrow  x_1 + 2x_2 + x_4  = 3 , \mspace{20mu} x_4 \geq 0 \\
\end{align}
$$


經過這些轉換後，<a href="#eq1">（公式一）</a> 變成 Standard Form ，如下：


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


可以再將它寫成矩陣形式：



$$
\begin{align}
&\mathbf{c} = \begin{bmatrix}
-2 \\
-3 \\
0 \\
0 \\
\end{bmatrix}
\mathbf{x} = \begin{bmatrix}
x_1 \\
x_2 \\
x_3 \\
x_4 \\
\end{bmatrix}
\mathbf{b} = \begin{bmatrix}
2 \\
3 \\
\end{bmatrix} \\
&\mathbf{A} = \begin{bmatrix}
1 &1 &1 &0 \\
1 &2 &0 &1 \\
\end{bmatrix} \\
\end{align}
$$


$$
\begin{align}
&\text{minimize: } z =\mathbf{c}^T\mathbf{x}  \\
&\text{subject to: } \mathbf{A}\mathbf{x} = \mathbf{b} \\
&\mathbf{x}\geq 0 \\
\end{align}
$$


至於如何得出 Linear Programming 的最佳解？會在後續文章講解。

## Reference 

本文參考交通大學開放式課程：線性規劃

http://ocw.nctu.edu.tw/course_detail.php?bgid=3&gid=0&nid=245#.WS2AasklFYc
