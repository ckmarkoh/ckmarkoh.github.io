---
layout: post
title: 'Ruby -- struct v.s openStruct'
date: 2014-03-12 15:26
comments: true
categories: ['Ruby Programming']
---


## 1.Struct


在 *Ruby* 程式語言裡面, 如果想要定義一個新的類別, 但又覺得重新寫一個 *class* 太麻煩, 

有比較方便的方法, 



```ruby
irb> Customer = Struct.new(:name, :phone) do
irb>   def say_hello
irb>    "Hello, I'm #{name}!"
irb>   end 
irb> end 
=> Customer
irb> Gary = Customer.new("Gary", "0923-355-599")
=> #<struct Customer name="Gary", phone="0923-355-599">
irb> Gary.phone
=> "0923-355-599"
irb> Gary.say_hello
=> "Hello, I'm Gary!"

```

<!--more-->


除此之外, 我們可以直接用 `==` 來比對兩個 *Struct* 是否相等



```ruby
irb> Judy1 = Customer.new("Judy", "0911-998-223")
=> #<struct Customer name="Judy", phone="0911-998-223">
irb> Judy2 = Customer.new("Judy", "0911-998-223")
=> #<struct Customer name="Judy", phone="0911-998-223">
irb> Judy1 == Judy2
=> true
irb> Judy1 == Gary
=> false

```


取得 *Attribute Value* 的方法



```ruby
irb> Gary[0]
=> "Gary"
irb> Gary[:name] 
=> "Gary"
irb> Gary["name"]
=> "Gary"

```


*enumerator* , 逐一列出 *Attribute Value* 的方法



```ruby
irb> Gary.each {|x| puts(x) }
Gary
0923-355-599
irb> Gary.each_pair {|name, value| puts("#{name} => #{value}") }
name => Gary
phone => 0923-355-599

```


轉成 *String* 或 *Array*



```ruby
irb> Gary.to_s
=> "#<struct Customer name=\"Gary\", phone=\"0923-355-599\">"
irb> Gary.to_a
=> ["Gary", "0923-355-599"]

```


## 2.OpenStruct


*OpenStruct* 又比 *Struct* 更有彈性, 因為它可以任意增加 *Attribute* , 不像 *Struct* 要先限制好有哪些 *Attribute* 



```ruby
irb> Jason = OpenStruct.new
=> #<OpenStruct>
irb> Jason.name = "Jason"
=> "Jason"
irb> Jason.phone = "0954-323-454"
=> "0954-323-454"
irb> Jason.age = "27"
=> "27"
irb> Jason
=> #<OpenStruct name="Jason", phone="0954-323-454", age="27">

```


除了可增加 *Attribute* 之外, 也可以刪除 *Attribute* 



```ruby
irb> Jason.delete_field("phone")
=> "0954-323-454"
irb> Jason
=> #<OpenStruct name="Jason", age="27">

```


也可轉成 *String* 或 *Hash*



```ruby
irb> Jason.to_s
=> "#<OpenStruct name=\"Jason\", age=\"27\">"
irb> Jason.to_h
=> {:name=>"Jason", :age=>"27"}
irb> Jason.marshal_dump()
=> {:name=>"Jason", :age=>"27"}

```


## 3.Reference


http://yafeilee.me/blogs/5246cc891563884b06000001

http://www.ruby-doc.org/stdlib-2.1.1/libdoc/ostruct/rdoc/OpenStruct.html

http://www.ruby-doc.org/core-2.1.1/Struct.html


