---
layout: post
title: 'word2vec (part 2 : Backward Propagation)'
date: 2016-07-12 09:21
comments: true
categories: ['Natural Language Processing','Neural Networks']
---

## Introduction


本文接續 [word2vec (part1)](/blog/2016/07/12/neural-network-word2vec-part-1-overview) ，介紹 *word2vec* 訓練過程的 *backward propagation* 公式推導。


*word2vec* 的訓練過程中，輸出的結果，跟上下文有關的字，在 *output layer* 輸出為 1 ，跟上下文無關的字，在 *output layer* 輸出為 0。 在此，把跟上下文有關的，稱為 *positive example* ，而跟上下文無關的，稱為 *negative example* 。


根據 [word2vec (part1)](/blog/2016/07/12/neural-network-word2vec-part-1-overview) 中提到的例子， *cat* 的向量為 $$\textbf{v}_2$$ ， *run* 的向量為 $$\textbf{w}_3$$ ， *fly* 的向量為 $$\textbf{w}_4$$ ，由於 *cat* 的上下文有 *run* ，所以 *run* 為 *positive example* ，而 *cat* 的上下文沒有 *fly* ，所以 *fly* 為 *negative example* ，如下圖所示：


![](/images/pic/pic_00187.png)


<!--more-->


## Objective Function


訓練類神經網路需要有個目標函數，如果希望 *positive example* 輸出為 1 ， *negative example* 輸出為 0，則可以將以下函數 $$J$$ 做最小化。


$$

J = - \text{log}(\frac{1}{1+e^{-\textbf{v}_I^T \textbf{w}_{pos} }}) - \sum_{neg} \text{log} ( 1- \frac{1}{1+e^{-\textbf{v}_I^T \textbf{w}_{neg} }} )

$$


其中 $$\textbf{v}_I$$ 為輸入端的字向量，而 $$\textbf{w}_{pos}$$ 和 $$\textbf{w}_{neg}$$ 為輸出端的字向量。 $$\textbf{w}_{pos}$$ 為 *positive example* ，而  $$\textbf{w}_{neg}$$ 為 *negative example* 。通常，對於每筆 $$\textbf{v}_I$$ 而言，會找一個 *positive example* 和多個 *negative example* ，因此用 $$\sum$$ 將這些 *negative example* 算出的結果給加起來。


先看這公式前半部的部分：


$$

-\text{log}(\frac{1}{1+e^{-\textbf{v}_I^T \textbf{w}_{pos} }})

$$


從以上公式得知，當 $$\frac{1}{1+e^{-\textbf{v}_I^T \textbf{w}_{pos} }} = 0$$ 時，  $$J$$ 會趨近無限大，而當 $$\frac{1}{1+e^{-\textbf{v}_I^T \textbf{w}_{pos} }} = 1$$ 時 ，  $$J$$ 會趨近 0 ，所以，降低 $$J$$ 的值，會迫使  $$\frac{1}{1+e^{-\textbf{v}_I^T \textbf{w}_{pos} }}$$ 接近 1 。


再來看另一部分：


$$

-\text{log}(1 - \frac{1}{1+e^{-\textbf{v}_I^T \textbf{w}_{neg} }})

$$


當 $$\frac{1}{1+e^{-\textbf{v}_I^T \textbf{w}_{neg} }} = 1$$ 時 $$J$$ 會趨近無限大，反之亦然，同理，降低 $$J$$ 的值，會迫使  $$1- \frac{1}{1+e^{-\textbf{v}_I^T \textbf{w}_{neg} }}$$ 接近 0 。


## Backward Propagation


至於要怎麼調整 $$\textbf{v}$$ 和  $$\textbf{w}$$ 的值，才能讓 $$J$$ 變小？ 就是要用到 *backward propagation* 。


### Positive Example


這邊先看 *positive example* 的部分， 輸入端為 *dog* ，輸出端為 *run* ，在進行偏微分公式推導之前，先定義一些符號，以便之後推導：


$$

\begin{align}

& x_{13} = \textbf{v}_1 \cdot \textbf{w}_3 \\

& y_{13} = \dfrac{1}{1+e^{-x_{13}}}  \\

& J = - \text{log} (y_{13} )

\end{align}

$$


這些符號，如下圖所示， $$x_{13}$$ 是 *output layer* 在 *sigmoid* 之前的值， $$y_{13} $$ 是通過 *sigmoid* 後的值， $$J$$ 是針對這筆 *positive example* 所算出來的 *cost function* 。


![](/images/pic/pic_00188.png)


如果要更新 $$\textbf{v}_1$$ 和 $$\textbf{w}_3$$ 的值，讓 $$J$$ 變小，就要用 *gradient descent* 的方式，過程如下：


$$

\begin{align}

& \textbf{v}_1 \leftarrow \textbf{v}_1 - \eta \nabla_{\textbf{v}_1}  J  \\

& \textbf{w}_3 \leftarrow \textbf{w}_3 - \eta \nabla_{\textbf{w}_1}  J   

\end{align}

$$


想瞭解更多關於 *gradient descent* ，請參考：[Gradient Descent & AdaGrad ](/blog/2015/12/23/optimization-method-adagrad)


其中， $$\eta$$ 為 *learning rate* ，為一常數，就是決定每一步要走多大，至於 $$\nabla_{\textbf{v}_1} J $$ 這項要怎麼算？


先看看它每個維度上的值：


$$

\nabla_{\textbf{v}_1} J = 

\begin{bmatrix}

\frac{\partial J}{\partial v_{11}} \\

\frac{\partial J}{\partial v_{12}} \\

\frac{\partial J}{\partial v_{13}} \\

\end{bmatrix}

$$



先看 $$\frac{\partial J}{\partial v_{11}}$$ 這項，可以用 *chain rule* 把它拆開：


$$

\frac{\partial J}{\partial v_{11} } = \frac{\partial J}{\partial y_{13}} \times \frac{\partial y_{13}}{\partial x_{13}}  \times \frac{\partial x_{13}}{\partial v_{11}}

$$


將 $$\frac{\partial J}{\partial v_{11} }$$ 拆成 $$\frac{\partial J}{\partial y_{13}}$$ 、 $$\frac{\partial y_{13}}{\partial x_{13}}$$ 和 $$\frac{\partial x_{13}}{\partial v_{11}}$$ 這三項。而這三項的值可分別求出來：


$$

\begin{align}

& \frac{\partial J}{\partial y_{13}} = \frac{\partial   - \text{log} (y_{13} )}{\partial y_{13}} = \frac{-1}{y_{13}} \\ 

& \frac{\partial y_{13}}{\partial x_{13}} = \frac{\partial (\frac{1}{1+e^{-x_{13}}})}{\partial x_{13}} = \frac{1}{1+e^{-x_{13}}}( 1- \frac{1}{1+e^{-x_{13}}}) = y_{13} ( 1- y_{13}) \\

& \frac{\partial x_{13}}{\partial v_{11}} = \frac{\partial \textbf{v}_{1} \cdot \textbf{w}_3} {\partial v_{11}} = w_{31}


\end{align}

$$


代回這三項的結果到 $$\frac{\partial J}{\partial v_{11} }$$ ，得出：


$$

\frac{\partial J}{\partial v_{11} } = \frac{-1}{y_{13}} \times y_{13} ( 1- y_{13}) \times  w_{31} = ( y_{13} - 1)  \times w_{31}

$$


而 $$\frac{\partial J}{\partial v_{12}}$$ 和 $$\frac{\partial J}{\partial v_{13}}$$ 也可用同樣方式得出其值， 如下：


$$

\nabla_{\textbf{v}_1} J = 

\begin{bmatrix}

\frac{\partial J}{\partial v_{11}} \\

\frac{\partial J}{\partial v_{12}} \\

\frac{\partial J}{\partial v_{13}} \\

\end{bmatrix}

=

\begin{bmatrix}

 ( y_{13} - 1)  \times w_{31} \\

 ( y_{13} - 1)  \times w_{32} \\

 ( y_{13} - 1)  \times w_{33} \\

 \end{bmatrix}

 = 

  ( y_{13} - 1)  \textbf{w}_3

$$



因此，可得出 $$\textbf{v}_1$$ 要調整的量：


$$

\textbf{v}_1 \leftarrow \textbf{v}_1 - \eta  ( y_{13} - 1)  \textbf{w}_3

$$


同理， $$\textbf{w}_3$$ 要調整的量：


$$

\textbf{w}_3 \leftarrow \textbf{w}_3 - \eta  ( y_{13} - 1)   \textbf{v}_1

$$


其中，可以把 $$( y_{13} - 1)$$ 看成是，模型預測出來的，和我們所想要的值，差距多少。因為在 *positive example* 的情況下，我們希望模型輸出結果 $$y_{13}$$ 為 1 。如果  $$y_{13} = 1$$ 表示模型預測對了，則不需要修正 $$\textbf{v}_1$$ 和 $$\textbf{w}_3$$ ，如果  $$y_{13} \neq 1$$ 時，才要修正 $$\textbf{v}_1$$ 和 $$\textbf{w}_3$$ 。


還有，之所以把這過程，稱為 *backward propagation* ，是因為可以把 *chain rule* 拆解 $$\nabla_{\textbf{v}_1} J $$ 的過程，看成是將 $$\frac{\partial J}{\partial y_{13} }$$ 的值， 由 *output layer* 往前傳遞，如下圖：


![](/images/pic/pic_00189.png)



想瞭解更多關於 *backward propagation* 的推導，請參考： [Backward Propagation 詳細推導過程 ](/blog/2015/05/28/neural-network-backward-propagation)


### Negative Example


再來看看 *negative example* 的部分， 輸入端為 *dog* ，輸出端為 *fly* ，在進行偏微分公式推導之前，先定義一些符號，以便之後推導：


$$

\begin{align}

& x_{14} = \textbf{v}_1 \cdot \textbf{w}_4 \\

& y_{14} = \dfrac{1}{1+e^{-x_{14}}}  \\

& J = - \text{log} (1 - y_{14} )

\end{align}

$$


這些符號，如下圖所示， $$x_{14}$$ 是 *output layer* 在 *sigmoid* 之前的值， $$y_{14} $$ 是通過 *sigmoid* 後的值， $$J$$ 是針對這筆 *positive example* 所算出來的 *cost function* 。


![](/images/pic/pic_00190.png)


同之前 *positive example* ，如果要更新 $$\textbf{v}_1$$ 和 $$\textbf{w}_4$$ 的值，讓 $$J$$ 變小，就要用 *gradient descent* 的方式：


$$

\begin{align}

& \textbf{v}_1 \leftarrow \textbf{v}_1 - \eta \nabla_{\textbf{v}_1}  J  \\

& \textbf{w}_4 \leftarrow \textbf{w}_4 - \eta \nabla_{\textbf{w}_1}  J   

\end{align}

$$


剩下的推導和 *positive example* 時，幾乎一樣，只有 $$J$$ 不一樣。此處只需推導 $$\frac{\partial J}{\partial y_{14}}$$ 的結果。


$$

\frac{\partial J}{\partial y_{14}} = \frac{\partial   - \text{log} (1 - y_{14} )}{\partial y_{14}} = \frac{1}{1 - y_{14}} 

$$


代回此結果到 $$\frac{\partial J}{\partial v_{11} }$$ ，得出：


$$

\frac{\partial J}{\partial v_{11} } = \frac{1}{ 1- y_{14}} \times y_{14} ( 1- y_{14}) \times  w_{41} = ( y_{14} - 0)  \times w_{41}

$$


於是可以得出要修正的量：


$$

\begin{align}

& \textbf{v}_1 \leftarrow \textbf{v}_1 - \eta  ( y_{14} - 0)  \textbf{w}_4 \\ 

& \textbf{w}_4 \leftarrow \textbf{w}_4 - \eta  ( y_{14} - 0)   \textbf{v}_1

\end{align}

$$


其中，可以把 $$( y_{14} - 0)$$ 看成是，模型預測出來的，和我們所想要的值，差距多少。因為在 *negative example* 的情況下，我們希望模型輸出結果 $$y_{14}$$ 為 0 。如果  $$y_{13} = 0$$ 表示模型預測對了，則不需要修正 $$\textbf{v}_1$$ 和 $$\textbf{w}_4$$ ，如果  $$y_{14} \neq 0$$ 時，才要修正 $$\textbf{v}_1$$ 和 $$\textbf{w}_4$$ 。


## Further Reading


關於如何從無到有，不使用任何自動微分的套件，實作一個 *word2vec* ，請看：[word2vec (part3)](/blog/2016/08/29/neural-network-word2vec-part-3-implementation)


註： 實際上， *word2vec* 的 *input layer* 和 *output layer* 各有兩種架構。*input layer* 有 *cbow* 和 *skip-gram* 兩種，*output layer* 有 *hierarchical sofrmax* 和 *negative sampling* 兩種，本文所寫的為 *skip-gram* 搭配 *negative sampling* 的架構。關於 *hierarchical softmax* 請參考[hierarchical sofrmax](/blog/2015/05/23/-hierarchical-probabilistic-neural-networks-neural-network-language-model) 。


