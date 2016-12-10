---
layout: post
title: 'Ruby -- Enumerable 1 : Collect , Inject '
date: 2014-04-30 15:06
comments: true
categories: [ruby, enumerable, map, reduce, functional_programming]
---

## Introduction


在 *ruby*  裡面, 也有類似 *functional programming style* 的東西, 像是 *map* , *reduce* 之類的


但是這些不是像 *python* 那樣是屬於 *build in function* , 像這樣



```python

>>> map(lambda x:x*2 ,[1,2,3,4])
[2, 4, 6, 8]

```


在 *ruby* 裡面的 *map* 是這樣, 如下



```ruby
irb(main):001:0> [1,2,3,4].map{|x| x*2}
=> [2, 4, 6, 8]

```


它是 *Enumerable Type* 的 *Method* 


除此之外, *map* 和 *reduce* 在 *ruby* 裡面還有其他的別名, 就是 *collect* 和 *inject*

<!--more-->

## collect and map


給定一個 *array*, `a1`



```ruby
irb(main):010:0> a1 = [1,2,3,4,5]
=> [1, 2, 3, 4, 5]

```


*collect* 和 *map* 的意思是一樣的

我們如果要把每個元素乘上三再加二, 可以用以下方法



```ruby
irb(main):011:0> a1.collect{|x| x*3+2 }
=> [5, 8, 11, 14, 17]
irb(main):012:0> a1.map{|x| x*3+2 }
=> [5, 8, 11, 14, 17]

```


與 *python* 不同的是, *python* 只可以寫 *in-line* 的 *anonymous function* , 如果要寫多行的, 就要重新寫個 `def function():`


*ruby* 可以直接寫多行的 *anonymous function* , 方法如下



```ruby
irb(main):004:0> a1.collect do |x|
irb(main):005:1* y = x+3
irb(main):006:1> x = y*2
irb(main):007:1> end
=> [8, 10, 12, 14, 16]
irb(main):008:0> a1.map do |x|
irb(main):009:1* y = x+3
irb(main):010:1> x = y*2
irb(main):011:1> end
=> [8, 10, 12, 14, 16]

```


## collect_concat and flat_map


如果我們要處理的是 *array of array*



```ruby
irb(main):012:0> a2 = [[-1, 5], [-2, 8], [-3, 11], [-4, 14], [-5, 17]]
=> [[-1, 5], [-2, 8], [-3, 11], [-4, 14], [-5, 17]]

```


希望做完運算以後可以把結果合併成同一個 *array* , 而不是*array of array* , 用 *collection* 就不夠了



```ruby
irb(main):013:0> a2.collect{|x| [-x,x*3+2] }
=> [[-1, 5], [-2, 8], [-3, 11], [-4, 14], [-5, 17]]

```


這時就要用 *collect_concat* 或 *flat_map*



```ruby
irb(main):014:0> a2.collect_concat{|x| [-x,x*3+2] }
=> [-1, 5, -2, 8, -3, 11, -4, 14, -5, 17]
irb(main):015:0> a2.flat_map{|x| [-x,x*3+2] }
=> [-1, 5, -2, 8, -3, 11, -4, 14, -5, 17]

```



## inject and reduce



假設有這樣一個 *string of array* , 從 '2' 到 '6' 



```ruby
irb(main):033:0> a3 = ['2','3','4','5','6']
=> ["2", "3", "4", "5", "6"]

```


如果我們希望把同一個 *array* 中所有的 *string* 都合併起來, 可以用 *+* 這個 *operation* , 進行 *reduce*, 或是寫 *anonymous function* 用 *inject*




```ruby
irb(main):034:0> a3.reduce(:+)
=> "23456"
irb(main):035:0> a3.inject { |sum, n| sum + n }
=> "23456"

```


如果起始的第一個 *element* , 不在 *array* 中, 例如 *1* 不在 `a3` 裡面, 則可以用以下方法



```ruby
irb(main):036:0> a3.reduce('1',:+)
=> "123456"
irb(main):037:0> a3.inject('1'){ |sum, n| sum + n }
=> "123456"

```


當然, *reduce* 和 *inject* 也可以寫成 *multiple line* 的 *anonymous function* , 如下



```ruby
irb(main):039:0> %w{ cat sheep bear }.reduce do |memo, word|
irb(main):040:1*    memo.length > word.length ? memo : word
irb(main):041:1> end
=> "sheep"
irb(main):042:0> %w{ cat sheep bear }.inject do |memo, word|
irb(main):043:1*    memo.length > word.length ? memo : word
irb(main):044:1> end
=> "sheep"

```


## Reference


http://www.ruby-doc.org/core-2.1.0/Enumerable.html