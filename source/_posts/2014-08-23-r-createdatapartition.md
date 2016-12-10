---
layout: post
title: 'R -- Data Splitting'
date: 2014-08-23 06:46
comments: true
categories: [data_splitting, Sample, createDataPartition, r]
---

在做機器學習演算法時，常常需要把資料分成 *training set* 和 *validation set* 這兩個資料組。

但是，要如何切割，才可以讓具有不同 *label* 的資料，在這兩個資料組中，平均分佈？


用統計語言 *R* 處理 *iris* 資料組為例。*iris* 的資料如下：


| index| Sepal.Length| Sepal.Width| Petal.Length| Petal.Width|   Species
| ---- | ----------- | ---------- | ----------- | ---------- | ----------
| 1    |        5.1  |       3.5  |        1.4  |       0.2  |   setosa
| 2    |        4.9  |       3.0  |        1.4  |       0.2  |   setosa
| ...  |        ...  |       ...  |        ...  |       ...  |   ...... 
| 51   |        7.0  |       3.2  |        4.7  |       1.4  |   versicolor
| 52   |        6.4  |       3.2  |        4.5  |       1.5  |   versicolor
| ...  |        ...  |       ...  |        ...  |       ...  |   ...... 
| 101  |        6.3  |       3.3  |        6.0  |       2.5  |   virginica
| 102  |        5.8  |       2.7  |        5.1  |       1.9  |   virginica
| ...  |        ...  |       ...  |        ...  |       ...  |   ...... 

<!--more-->


*iris* 的中文意思是 *鳶尾花* 。這筆資料中，給出了 *Sepal* (萼片）和 *Petal* （花瓣）的長寬，以及 *Species* （物種）。有了這筆資料，就可以把花瓣和萼片的長度，當成是 *feature* ，而把物種當成 *label* ，就可以用機器學習的演算法，以花瓣和萼片的長度，來推測物種是什麼。


首先，到 *R console* 中，直接輸入 `iris` ，就會顯示出以下資料，如下：



```R

> iris
    Sepal.Length Sepal.Width Petal.Length Petal.Width    Species
1            5.1         3.5          1.4         0.2     setosa
2            4.9         3.0          1.4         0.2     setosa
3            4.7         3.2          1.3         0.2     setosa
4            4.6         3.1          1.5         0.2     setosa
5            5.0         3.6          1.4         0.2     setosa
6            5.4         3.9          1.7         0.4     setosa
7            4.6         3.4          1.4         0.3     setosa
8            5.0         3.4          1.5         0.2     setosa

```


用 `table` 指令，可以統計一下每個物種的個數，各有幾種。



```R

> table(iris$Species)

    setosa versicolor  virginica 
        50         50         50 

```


在這資料組中，共有三個物種，每個物種都有50筆資料。由於每種物種的個數比為1:1:1，希望在切割完後的 *training set* 和 *validation set* 中，各物種的個數比也都為1:1:1。


由於這個資料已經先根據 *species* 排序過了，不可以直接取一個 *index* 把資料切成兩半。

如下，假設要取前120筆資料作為*training set* 而後30筆資料作為 *validation set* ，語法如下。



```R

> trainSet <- iris[1:120,]
> valSet <- iris[121:150,]

```


用 `table` 來統計這兩組資料中 *species* 的分佈，會發現以下情形：



```R

> table(trainSet$Species)

    setosa versicolor  virginica 
        50         50         20 

> table(valSet$Species)

    setosa versicolor  virginica 
         0          0         30 

```


在 `trainSet` 中，大部分的 *species* 為 *setosa* 和 *versicolor* ，而在 `valSet` 中，只有 *virginica* ，各物種的分佈不平均。


## Split by sample


`sample` 是隨機在一個數字範圍中，取出任意數字（不重複或可重複）。例如在 1~10 中隨機取五個不重複的數字，可用以下方法：



```R

> sample(1:10, size=5)
[1] 8 7 9 3 4

```


由於 `sample` 產生的結果是隨機的，所以要先設定一下 *seed* ，以便讓每次產生的結果都一樣。



```R

> set.seed(123456)

```


可以隨機在 *iris* 的 *index* 範圍中，隨機取出 120 個數字作為 *training set* 的 *index*，而剩下沒取到的 30 個數字的作為 *validation set* 的 *index* ，作法如下：



```R

> trainIndex <- sample(nrow(iris), size=120)
> trainSet <- iris[trainIndex,]
> valSet <- iris[-trainIndex,]

```


其中， `trainIndex` 是要取出來作為 *training set* 的 *index* 。用 `table` 後統計結果如下：



```R

> table(trainSet$Species)

    setosa versicolor  virginica 
        41         35         44 

> table(valSet$Species)

    setosa versicolor  virginica 
         9         15          6 

```


看起來分佈比較平均了，但是仍有點誤差，還不是1:1:1。


## Split by createDataPartition


如果要讓各個物種能夠在 *training set* 及 *validation set* 中，達到1:1:1的平均分佈，可以用 `createDataPartition`。


首先，要載入模組 `caret`。



```R

> library(caret)

```


用 `createDataPartition` 根據 *iris* 的 *species* 種類，隨機取出120筆資料，作為 *training set* 的 *index* ，作法如下：



```R

> trainIndex <- createDataPartition(iris$Species, p = .8, list=FALSE)
> trainSet <- iris[trainIndex,]
> valSet <- iris[-trainIndex,]

```


其中， `iris$Species` 表示要根據 *iris* 的 *species* 來取 *index*， 而 `p = .8` 表示要取出 0.8*150 筆資料，而 `list=FALSE` 表示產生出的結果不是 *list* ，而是 *vector* 。


用 `table` 來統計一下分割的結果，物種以1:1:1的平均分佈在 `trainSet` 和 `valSet` 。



```R

> table(trainSet$Species)

    setosa versicolor  virginica 
        40         40         40 

> table(valSet$Species)

    setosa versicolor  virginica 
        10         10         10 

```


## Reference


關於 createDataPartition 可參考以下網址：

http://www.inside-r.org/packages/cran/caret/docs/createDataPartition


關於 Data splitting 的其他方法，可參考以下網址：

http://topepo.github.io/caret/splitting.html

