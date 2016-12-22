---
layout: post
title: 'Hello World'
date: 2014-02-15 18:04
comments: true
published: false 
---

Hi, This a demo post of [Logdown](http://logdown.com). 


Logdown use Markdown as main syntax, you can find more example by reading this [document on Wikipedia](http://en.wikipedia.org/wiki/Markdown)


Logdown also support drag & drop image uploading. The picture syntax is like this:


![](/images/pic/pic_00000.png)


## Bloging with code snippet:


`inline code`


### Plain Code



```linenos:false 
puts "Hello World!"

```


### Code with Language



```ruby
puts "Hello World!"

```


### Code with Title



```ruby hello_world.rb
puts "Hello World!"

```



## MathJax Example


### Mathjax


$$

x = \dfrac{-b \pm \sqrt{b^2 - 4ac}}{2a}

\\\\

	\newcommand\T{\Rule{0pt}{1em}{.3em}}

	\begin{array}{|c|c|}

	\hline X & P(X = i) \T \\\hline

	  1 \T 

    \begin{array}{|c|c|}

    \hline X & P(X = i) \T \\\hline

    \hline X & P(X = i) \T \\\hline

		\end{array}

    \\\hline

	  2 \T & 1/6 \\\hline

	  3 \T & 1/6 \\\hline

	  4 \T & 1/6 \\\hline

	  5 \T & 1/6 \\\hline

	  6 \T & 1/6 \\\hline

	\end{array}



$$


$$

\begin{array}{|c|}

\hline y \\ \hline

\\

  \begin{array}{|c|}

     \hline x  \\ \hline

      woman(x) \\

     \hline

   \end{array}

    \Rightarrow

  \begin{array}{|c|}

     \hline \\[8pt]  \hline

      walk(x) \\

     \hline

   \end{array}

\\

y=? \\

smoke(y) \\

\hline

\end{array}

$$


### Inline Mathjax


The answser is $$a^2 + b^2 = c^2$$.


## Table Example


| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 1         | Hello         | $1600 |
| col 2         | Hello         |   $12 |
| col 3         | Hello         |    $1 |



--------


## multiple line bracket

$$


\begin{cases} 

1 & x>1 \\

0 & \text{otherwise}

\end{cases}

$$


## align

$$

\begin{align}

&x=1  &y=1 \\

&x=2  &y=2

\end{align}

$$


$$

\exists e(butter(e), agent(e,John), theme(e,toast))

$$


*Logistic Regression* 的 *cost function* 如下：

$$

J(\theta) = \frac{-1}{m}[\sum_{i}^{m}y^{(i)}log( h_{\theta}(x^{(i)})) + (1-y^{(i)})log (1 -h_{\theta}(x^{(i)} ) ) ]

$$


而類神經網路的 *cost function* 如下：


$$

J(\theta) = \frac{-1}{m}[\sum_{i}^{m}\sum_{k}^{K}y_{k}^{(i)}log( h_{\theta}(x^{(i)})_{k}) + (1-y_{k}^{(i)})log (1 -h_{\theta}(x^{(i)} )_{k} ) ]

$$



$$

\begin{bmatrix}

r_{0}  \\[0.3em]

r_{1}  \\[0.3em]

r_{2}  \\[0.3em]

\end{bmatrix}

$$



test line1: $$ \mid A\mid  $$

test line2: $$ p(A\mid B) $$

test line3: $$ KL(A\mid \mid B) $$

