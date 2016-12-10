---
layout: post
title: 'Note on Xpath Parsing'
date: 2014-03-13 06:45
comments: true
categories: [web_crawler, xPath]
---

本篇是我在使用xpath中遇到的一些案例,做些筆記


xpath是什麼,簡而言之,就是可以把網頁的html碼當成Tree

並且用xpath去取得Tree中的某些nodes或leaves

若想了解更多關於xpath是什麼,可以考以下網站

http://www.w3schools.com/XPath/default.asp


以下的xpath實作是以jquery的$$x("xpath")`經由chrome brwser的console來操作

至於xpath的實際應用,可以用在web crawler中從網頁中擷取想要的東西

這樣就不需要自己寫個parser來處理html了

<!--more-->

##1.following-sibling, preceding-sibling:


首先來看以下這種情況



```html example1.html
1<br> 2<br> 3 <br> 4 

```


這些數字是和br相鄰,而不是被它給包住

這時就要用到sibling來把數字取出



```javascript

> $x("//br[3]/following-sibling::text()")
[" 4 "]

> $x("//br[2]/following-sibling::text()")
[ " 3 " , " 4 " ]

> $x("//br[1]/following-sibling::text()")
[" 2 ", " 3 " , " 4 " ]

> $x("//br[2]/preceding-sibling::text()")
[" 2 ", " 1 "]

```


也可以把text和br的位置互換



```

> $x("//text()[following-sibling::br[3]]")
[ " 1 " ]

> $x("//text()[following-sibling::br[2]]")
[ " 1 " , " 2 " ]

> $x("//text()[following-sibling::br[1]]")
[ " 1 " , " 2 " , " 3 " ]

> $x("//text()[preceding-sibling::br[2]]")
[ " 3 " , " 4 " ]

```


##2.table position() , last()


如果我們要取出以下表格中的某幾個column



```html example2.html
<table>
    <tbody>
         <tr> <td>11</td><td>12</td><td>13</td><td>14</td><td>15</td><td>16</td> </tr>
        <tr> <td>21</td><td>22</td><td>23</td><td>24</td><td>25</td><td>26</td> </tr>
    </tbody>
<table>

```


可以用這個方法



```javascript

> $x("//table/tbody/tr/td[last()]/text()")
[ "16" , "26" ]

> $x("//table/tbody/tr/td[last()-1]/text()")
[ "15" , "25" ]

> $x("//table/tbody/tr/td[position() mod 3 = 1]/text()")
[ "11" , "14" , "21" , "24" ]

> $x("//table/tbody/tr/td[position() mod 3 = 2]/text()")
[ "12" , "15" , "22" , "25" ]

```


##結語


以上是很簡單的幾個例子

除了使用jquery以外,python也有提供xpath的library

例如lxml

可參考以下這篇

http://lxml.de/xpathxslt.html
