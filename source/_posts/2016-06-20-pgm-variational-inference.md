---
layout: post
title: '機率圖模型 -- Variational Inference'
date: 2016-06-20 02:42
comments: true
categories: 
---

## Introduction


若某個有 $$m$$ 個維度的 data $$\textbf{x} = \{x_1,x_2,...,x_m\}$$ 的產生，是跟某個有 $$n$$個維度的 hidden variable $$\textbf{z} = \{z_1,z_2,...,z_n\} $$ 有關，在機率圖模型，表示成：  


![](/images/pic/pic_00171.png)


從這機率圖形，可得出 hidden variable $$\textbf{z}$$ 和 $$\textbf{x}$$ 的聯合分佈機率：


$$

p(\textbf{x},\textbf{z})  = p(\textbf{z})p(\textbf{x}|\textbf{z})

$$


若是給定 hidden variable $$\textbf{z}$$ 的值，則可產生 data $$\textbf{x}$$ ，如下：


$$

p(\textbf{x}|\textbf{z}) 

$$


但如果給定 data $$\textbf{x}$$ 的值，這組 data 所對應到的 hidden variable 的值，如下：


$$

p(\textbf{z}|\textbf{x}) = \frac{p(\textbf{x} , \textbf{z})}{p(\textbf{x})}  =\frac{p(\textbf{x} | \textbf{z}) p(\textbf{z}) }{\int p(\textbf{x} | \textbf{z})p(\textbf{z}) \text{d}\textbf{z}} 


$$


其中，分母 $$\int p(\textbf{x} \mid  \textbf{z})p(\textbf{z}) \text{d}\textbf{z}$$ 積分如果無法算出來的時候，就無法直接算出 $$p(\textbf{z}\mid \textbf{x})$$ 的值，則要用估計的方法來計算 $$p(\textbf{z}\mid \textbf{x})$$ 的值。


Variational Inference 用來估計 $$p(\textbf{z}\mid \textbf{x})$$ 的值 。


<!--more-->


*Variational Inference* 的做法是，不直接把 $$p(\textbf{z}\mid \textbf{x})$$ 求出，而是用一個較好算的 $$q(\textbf{z}\mid \mathbf{\theta})$$ 來求出近似解。其中， $$\mathbf{\theta}$$ 為參數，調整此參數可以讓 $$q(\textbf{z}\mid \mathbf{\theta})$$。較接近 $$p(\textbf{z}\mid \textbf{x})$$ 。


求近似解的方法，是讓 $$q(\textbf{z}\mid \mathbf{\theta})$$ 和 $$p(\textbf{z}\mid \textbf{x})$$ 這兩個機率分佈的 *KL Divergence* 越小越好：<a name="eq1">（公式一）</a>


$$

\min_{\theta} D_{KL}[  q(\textbf{z}| \mathbf{\theta} )  ||   p(\textbf{z}|\textbf{x})   ]

$$


## Evidence Lower Bound (ELOB)


由於<a href="#eq1">（公式一）</a>中的 $$p(\textbf{z}\mid \textbf{x})$$ 無法直接計算出，因此這個

*KL Divergence* 的值也無法算出來。但可以把它稍微整理一下，如下<a name="eq2">（公式二）</a>：


$$

\begin{align}


&

D_{KL}[q(\textbf{z})||p(\textbf{z}|\textbf{x})] = \int

q(\textbf{z}) \text{log}  \frac{q(\textbf{z})}{p(\textbf{z}|\textbf{x})} \text{d}\textbf{z}

\\&


=  \int

q(\textbf{z}) \text{log} \frac{  q(\textbf{z}) p(\textbf{x}) }{ p(\textbf{x},\textbf{z})  } \text{d}\textbf{z}

\\

&


=  \int

q(\textbf{z}) \text{log} \frac{  q(\textbf{z})  }{ p(\textbf{x},\textbf{z})  } \text{d}\textbf{z}

+ \int q(\textbf{z})\text{log} {p(\textbf{x})} \text{d}\textbf{z}

\\


&

=  \int 

q(\textbf{z})

\bigg(\text{log}   q(\textbf{z})

-\text{log} p(\textbf{z},\textbf{x} )\bigg)  \text{d} \textbf{z}

+ \text{log} p(\textbf{x})

\\

&

= -\bigg( E_{q(\textbf{z}) }[\text{log}p(\textbf{x},\textbf{z})] - E_{ q(\textbf{z} ) }[\text{log}q(\textbf{z})]\bigg) + \text{log}p(\textbf{x})


\end{align}

$$


註：以上省略 $$\theta$$


定義 Evidence Lower Bound (ELOB) $$L[q(\textbf{z})]$$ 為：


$$

L[q(\textbf{z})]

   =  E_{q(\textbf{z}) }[\text{log}p(\textbf{x},\textbf{z})] - E_{ q(\textbf{z} ) }[\text{log}q(\textbf{z})]


$$


將 $$L[q(\textbf{z})]$$ 代入<a href="#eq2">（公式二）</a>的推導結果，得出：


$$

D_{KL}[q(\textbf{z})||p(\textbf{z}|\textbf{x})]  = -L[q(\textbf{z})]

 + \text{log}p(\textbf{x})

$$


因此：


$$

\text{log}p(\textbf{x})  = D_{KL}[q(\textbf{z})||p(\textbf{z}|\textbf{x})] +  L[q(\textbf{z})]

$$


由於 $$p(\textbf{x})$$ 為一個固定的機率分佈，因此 $$\text{log}p(\textbf{x})$$ 為常數。


因此，只要將 $$L[q(\textbf{z})]$$ 的最大化，即可將 $$D_{KL}[q(\textbf{z})\mid \mid p(\textbf{z}\mid \textbf{x})] $$ 最小化。


而 $$L[q(\textbf{z})]$$ 中的 $$p(\textbf{x}, \textbf{z})$$ 是可以算得出來的，因此目標函數為：


$$

\max_{\theta} L[q(\textbf{z} | \theta )]


$$


## Mean-Field Variational Inference


*Variational Inference* 有很多種作法，其中一種常見的作法為 *Mean-Field Variational Inference* ，這種方法是，假設 $$q(\textbf{z}\mid  \mathbf{\theta} )$$ 中的每個維度都是獨立的。這樣會比較容易求出它的值，即：<a name="eq3">（公式三）</a>


$$

q(\textbf{z}|\mathbf{\theta}) = \prod_{i} q_{i}(z_{i}| \theta_{i} ) 

$$


每個獨立的 $$q(z_{i}\mid \theta)$$ 都是機率分佈，即積分結果為1：


$$

\forall i  ,  \int q_{i}(z_{i}| \theta_{i} )  \text{d} z_{i} = 1

$$



## Maximizing ELOB


再來，用 *Mean-Field Variational Inference* 的方法，把  $$L[q(\textbf{z})]$$  整理一下。


 $$L[q(\textbf{z})]$$ 可分成兩部分：$$ E_{q(\textbf{z}) }[\text{log}p(\textbf{x},\textbf{z})]$$ 和 $$  E_{ q(\textbf{z} ) }[\text{log}q(\textbf{z})] $$


現在，要分別把<a href="#eq3">（公式三）</a>代入這兩項，並稍做整理，讓它們的值可以被求出來。


首先，來看 $$ E_{q(\textbf{z}) }[\text{log}p(\textbf{x},\textbf{z})]$$ ：


$$

\begin{align}

& E_{q(\textbf{z}) }[\text{log}p(\textbf{x},\textbf{z})] = \int \prod_{i} q_{i}(z_{i}) \text{log} p (\textbf{z},\textbf{x}) \text{d}\textbf{z}

\\

&

= \int_{z_{1}}\int_{z_{2}} ... \int_{z_{n}} \int \prod_{i} q_{i}(z_{i}) \text{log} p (\textbf{z},\textbf{x}) \text{d}z_{1} \text{d}z_{2}...\text{d}z_{n}

\end{align}

$$


挑出 $$\textbf{z}$$ 中的某個元素： $$z_{j}$$ ，整理一下，繼續以上推導過程：


$$

\begin{align}

& = \int_{z_{j}} q_{j}(z_{j}) \bigg( \int... \int_{z_{i\neq j }} \prod_{i} q(z_{i}) \text{log} p (\textbf{z},\textbf{x}) \prod_{i \neq j} \text{d}z_{i} \bigg) \text{d}z_{j}

\\

& =  \int_{z_{j}} q_{j}(z_{j}) \bigg( \int... \int_{z_{i\neq j }} \text{log} p (\textbf{z},\textbf{x}) \prod_{i \neq j } q(z_{i})  \text{d}z_{i} \bigg) \text{d}z_{j}

\end{align}

$$


由於 $$ \prod_{i \neq j } q(z_{i})$$ 是 joint distribution，可以把 $$p (\textbf{z},\textbf{x})$$ 放到此機率分布的期望值裡，繼續以上推導過程：<a name="eq4">（公式四）</a>


$$

=  \int q_{j}(z_{j}) E_{q_{i \neq j}(z)}\bigg[  \text{log} p (\textbf{z},\textbf{x}) \  \bigg] \text{d}z_{j}

$$


由於 $$E_{q_{i \neq j}(z)}\bigg[  \text{log} p (\textbf{z},\textbf{x}) \  \bigg]$$ 中，把 $$p (\textbf{z},\textbf{x})$$ 中不是 $$z_{j}$$ 的 $$z_{i}$$ 全都消掉了，所以算完後的 $$p (\textbf{z},\textbf{x})$$ 只會剩 $$z_{j}$$ ，令：


$$

\text{log} \tilde{p}(\textbf{x},z_{j}) =  E_{q_{i \neq j}(z)}\bigg[  \text{log} p (\textbf{z},\textbf{x}) \  \bigg] 

$$


將其代入<a href="#eq4">（公式四）</a>，繼續推導過程，得出<a name="eq5">公式五</a>：


$$

 E_{q(\textbf{z}) }[\text{log}p(\textbf{x},\textbf{z})]  =  \int q_{j}(z_{j})   \text{log} \tilde{p}(\textbf{x},z_{j}) \text{d}z_{j}

$$


再來看 $$ E_{ q(\textbf{z} ) }[\text{log}q(\textbf{z})]$$：


$$

\begin{align}

& E_{ q(\textbf{z} ) }[\text{log}q(\textbf{z})] = \int \prod_{i} q_{i}(z_{i}) \text{log} \prod_{i} q (z_{i}) \text{d}\textbf{z}

\\

&

= \int \prod_{i} q_{i}(z_{i}) \sum_{i} \text{log} q (z_{i}) \text{d}\textbf{z}

\\

&

= \sum_{i} \bigg(\prod_{k \neq i} \int_{k}q_{k}(z_{k})\text{d}z_{k}  \int  q_{i}(z_{i}) \text{log} q (z_{i}) \text{d}z_{i}  \bigg)

\\

&

= \sum_{i} \bigg(  \int  q_{i}(z_{i}) \text{log} q (z_{i}) \text{d}z_{i}  \bigg)

\end{align}

$$


挑出 $$\textbf{z}$$ 中的某個元素： $$z_{j}$$ ，整理一下，繼續以上推導過程：


$$

=  \int  q_{j}(z_{j}) \text{log} q (z_{j}) \text{d}z_{j} + \sum_{i \neq j } \bigg(  \int  q_{i}(z_{i}) \text{log} q (z_{i}) \text{d}z_{i}  \bigg)

$$


不涉及 $$j$$的部分，就當 $$const$$ 來處理，整理以上公式，得出：<a name="eq6">（公式六）</a>


$$

E_{ q(\textbf{z} ) }[\text{log}q(\textbf{z})] =  \int  q_{j}(z_{j}) \text{log} q (z_{j}) \text{d}z_{j} + const

$$


將 <a href="#eq5">（公式五）</a>和<a href="#eq6">（公式六）</a> 代入 $$L[q(\textbf{z})]$$ ，得出：


$$

\begin{align}

& L[q(\textbf{z})] =\int q_{j}(z_{j})   \text{log} \tilde{p}(\textbf{x},z_{j}) \text{d}z_{j}- \int  q_{j}(z_{j}) \text{log} q (z_{j}) \text{d}z_{j} - const

\\

&

= \int q_{j}(z_{j})   \text{log} \frac{ \tilde{p}(\textbf{x},z_{j}) } { q_{j}(z_{j}) } \text{d} z_{j} - const

\\

& 

= -D_{KL}[\tilde{p}(\textbf{x},z_{j})|| q_{j}(z_{j} ) ] - const

\end{align}

$$


根據以上結果，要將 $$ L[q(\textbf{z})] =\int q_{j}(z_{j}) $$ 最大化，則：


$$

D_{KL}[\tilde{p}(\textbf{x},z_{j})|| q_{j}(z_{j} ) ]  = 0

$$


因此：


$$

q_{j}(z_{j}) = \tilde{p}(\textbf{x},z_{j}) 

$$


由於 $$z_{j}$$ 有 $$n$$ 個， 整個運算過程，有點類似 *EM* 演算法，就是挑選某個 $$q_{j}(z_{j})$$ 根據以上公式，來更新其值，而其他的 $$q_{i}(z_{i})$$ 則固定。這樣依序更新每個 $$q_{j}(z_{j})$$ ，一直循環下去，則最後收斂的結果，即為 $$L[q(\textbf{z})]$$ 的 *Local Maximum* 。


## Reference


A Tutorial on Variational Bayesian Inference

http://www.orchid.ac.uk/eprints/40/1/fox_vbtut.pdf
