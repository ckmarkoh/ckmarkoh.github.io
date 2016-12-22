---
layout: post
title: 'Python argparse'
date: 2014-03-31 05:31
comments: true
categories: ['Python Programming']
---


## 0.Introduction


argparse是用來處理command line的argument所使用的

例如執行一個程式, 比如說 `mv`  

在Terminal輸入 `mv --help` 之後

顯示如下：



```sh
$ mv --help
Usage: mv [OPTION]... [-T] SOURCE DEST
  or:  mv [OPTION]... SOURCE... DIRECTORY
  or:  mv [OPTION]... -t DIRECTORY SOURCE...
Rename SOURCE to DEST, or move SOURCE(s) to DIRECTORY.

Mandatory arguments to long options are mandatory for short options too.
      --backup[=CONTROL]       make a backup of each existing destination file
  -b                           like --backup but does not accept an argument
  -f, --force                  do not prompt before overwriting
  -i, --interactive            prompt before overwrite
......

```


由此可知, `mv` 後面可以有很多種argument

那要寫程式去parse這些command line的argument很麻煩

如果用了 `argparse` 這個module

就不用自己寫程式去parse這些argument了

<!--more-->

接下來實作看看

首先, 寫個很簡單的程式, 試試看 `argparse`



```python argprog0.py
import argparse
parser = argparse.ArgumentParser(prog='argprog0')
args = parser.parse_args()
print args

```


其中, `prog='argprog0'` 為 *help* 的內容中,顯示的程式名稱

而 `args` 為 `argparse` parse完後的argument


這個程式可以判斷後面接的argument是不是 `-h` 或 `--help`

如果是的話, 則顯示出 *help* 的內容



```sh
$ python argprog0.py -h
usage: argprog0 [-h]

optional arguments:
  -h, --help  show this help message and exit

```


```sh
$ python argprog0.py --help
usage: argprog0 [-h]

optional arguments:
  -h, --help  show this help message and exit

```


如果輸入的argument不是`-h` 或 `--help`

則會顯示出error message



```sh
$ python argprog0.py 1
usage: argprog0 [-h]
argprog0: error: unrecognized arguments: 1

```


如果沒有輸入argument, 則顯示如下：



```sh
$ python argprog0.py 
Namespace()

```


其中, `Namespace()` 是 `args` 中的argument

因為沒有輸入任何argument, 所以 `Namespace()` 是空的

接下來,看看要怎麼把argument存到 `args` 裡面


## 1.Optional Argument


先來試試看加個 *optional argument* 到程式當中

所謂 *optional argument* 就是可有可無的argument

通常會以 `-` 或 `--` 開頭, 例如 `--help`

試試看加一個 `--foo` 到程式裡面 



```python argprog1.py
import argparse
parser = argparse.ArgumentParser(prog='argprog1')
parser.add_argument('--foo', help='foo of the %(prog)s program')
args = parser.parse_args()
print args

```


其中 `help='foo of the %(prog)s program'` 為 *help* 中, 對於 `--foo` 的解說

程式寫好之後, 先來看 `help` 有什麼改變



```sh
$ python argprog1.py -h
usage: argprog1 [-h] [--foo FOO]

optional arguments:
  -h, --help  show this help message and exit
  --foo FOO   foo of the argprog1 program

```


接著, 來輸入 `--foo` 的值

輸入方法如下, 在 `--foo` 後面接的argument就是它的值



```sh
$ python argprog1.py --foo FOO
Namespace(foo='FOO')

```


```sh
$ python argprog1.py --foo BAR
Namespace(foo='BAR')

```


也可以什麼都不輸入, 因為 `--foo` 是 *optional argument*

這樣的話 `foo` 的值就會是 `None`



```sh
$ python argprog.py 
Namespace(foo=None)

```


但是如果只輸入 `--foo` 而後面沒有值, 或是多輸入了一個值

皆會出現error



```sh
$ python argprog1.py --foo 
usage: argprog1 [-h] [--foo FOO]
argprog1: error: argument --foo: expected one argument

```


```sh
$ python argprog1.py --foo FOO BAR
usage: argprog1 [-h] [--foo FOO]
argprog1: error: unrecognized arguments: BAR

```



## 2.Positional Argument


再來, 於程式中加入 *positional argument*

和 *optional argument* 不一樣的地方在於

*positional argument* 是必須要輸入的argument

且 *positional argument* 的值, 是根據argument的位置而定

*positional argument* 的開頭, 沒有 `-` , 或 `--`


把 `bar1` 和 `bar2` 這兩個 *positional argument* 加到程式中,如下：



```python argprog2.py
import argparse
parser = argparse.ArgumentParser(prog='argprog2')
parser.add_argument('--foo', help='foo of the %(prog)s program')
parser.add_argument('bar1', help='bar1 of the %(prog)s program')
parser.add_argument('bar2', help='bar2 of the %(prog)s program')
args = parser.parse_args()
print args

```


程式寫好之後, 先來看 `help` 有什麼改變



```sh
$ python argprog2.py -h
usage: argprog2 [-h] [--foo FOO] bar1 bar2

positional arguments:
  bar1        bar1 of the argprog2 program
  bar2        bar2 of the argprog2 program

optional arguments:
  -h, --help  show this help message and exit
  --foo FOO   foo of the argprog2 program

```


接著輸入 *positional argument* 的值

輸入的時候, 不需要輸入argument的名稱

只要根據argument的位置, 依序輸入值就可以了

輸入的位置不同, argument的值就會不同, 如下：



```sh
$ python argprog2.py x y 
Namespace(bar1='x', bar2='y', foo=None)

```


```sh
$ python argprog2.py y x
Namespace(bar1='y', bar2='x', foo=None)

```


另外, 也可以輸入 *optional argument*

可以在任意位置插入 *optional argument* 

不會影響 *positional argument* 的值



```sh
$ python argprog2.py x y --foo z
Namespace(bar1='x', bar2='y', foo='z')

```


```sh
$ python argprog2.py x --foo z y
Namespace(bar1='x', bar2='y', foo='z')

```


```sh
$ python argprog2.py --foo z x y
Namespace(bar1='x', bar2='y', foo='z')

```


以下為出現error的情形

因為*positional argument* 是必須要輸入的

沒有輸入的話, 就會出現error message



```sh
$ python argprog2.py 
usage: argprog2 [-h] [--foo FOO] bar1 bar2
argprog2: error: too few arguments

```


```sh
$ python argprog2.py x
usage: argprog2 [-h] [--foo FOO] bar1 bar2
argprog2: error: too few arguments

```


```sh
$ python argprog2.py x --foo y
usage: argprog2 [-h] [--foo FOO] bar1 bar2
argprog2: error: too few arguments

```



## 3.Description and Epilog


如果我們要在 *help* 中, 加入關於整個程式的解說,

而不是針對個別的 argument做解說

可以用 `description` 和 `epilog` , 方法如下：



```python argprog3.py
import argparse
parser = argparse.ArgumentParser(
        prog='argprog3',
        description='### This is description. ###',
        epilog='### This is epilog. ###'
        )
parser.add_argument('--foo', help='foo of the %(prog)s program')
parser.add_argument('bar', help='bar of the %(prog)s program')
args = parser.parse_args()
print args

```


其中, `description` 和 `epilog` 為解說內容, 唯出現的位置不同

來看輸入 `-h` 後, 它們的位置在哪：



```sh
$ python argprog3.py -h
usage: argprog3 [-h] [--foo FOO] bar

### This is description. ###

positional arguments:
  bar         bar of the argprog3 program

optional arguments:
  -h, --help  show this help message and exit
  --foo FOO   foo of the argprog3 program

### This is epilog. ###

```


所以, description是在arguments解說的前方的內容

而epilog是在arguments解說的後方的內容


## 4.Default Value


針對 *optional argument* 在沒有輸入值得情形下

如果我們想要讓這個有 *Default* 的值, 而不是 *None*

可以用 `default` ,方法如下：



```python argprog4.py
import argparse
parser = argparse.ArgumentParser( prog='argprog4')
parser.add_argument('--foo', default='10',
                    help='foo of the %(prog)s program, (default: %(default)s)')
print parser.parse_args()

```


其中 `help='foo of the %(prog)s program, (default: %(default)s)'` 

是 *help* 中顯示的 *default* 值

然後輸入 `-h` 看看, help中有顯示出 default 的值是多少



```sh
$ python argprog4.py -h
usage: argprog4 [-h] [--foo FOO]

optional arguments:
  -h, --help  show this help message and exit
  --foo FOO   foo of the argprog4 program, (default: 10)

```


接著我們可以什麼都不輸入

看看 `foo` 的值是不是 *default* 的值



```sh
$ python argprog4.py 
Namespace(foo='10')

```


或者在 `--foo` 後面接其他值, 改變它的值



```sh
$ python argprog4.py --foo 29
Namespace(foo='29')

```


但是 `--foo` 後面沒有值的話, 還是會有Error



```sh
$ python argprog4.py --foo
usage: argprog4 [-h] [--foo FOO]
argprog4: error: argument --foo: expected one argument

```


## 5.Action


上一節提到, 如果 *optional argument* , 例如 `--foo`

後面沒有值的話, 還是會有Error

那麼像是 `--help` 這種 *optional argument*, 後面沒有接值, 也沒出現error

這是怎麼辦到的呢？

這就要用到 `action` 了, 例如以下程式中, 

`action='store_true'` 表示有輸入 `--foo1` 則 `foo1` 為 *true* 若沒有則為 *Fales*

`action='store_false'` 表示有輸入 `--foo2` 則 `foo2` 為 *Fales* 若沒有則為 *True*



```python argprog5.py
import argparse
parser = argparse.ArgumentParser( prog='argprog5')
parser.add_argument('--foo1', action='store_true', help='foo1 of the %(prog)s program')
parser.add_argument('--foo2', action='store_false', help='foo2 of the %(prog)s program')
args = parser.parse_args()
print args

```


接著,看看什麼都沒輸入的情形：



```sh
$ python argprog5.py 
Namespace(foo1=False, foo2=True)

```


再來是有輸入 `--foo1` 或 `--foo2` 的情形：



```sh
$ python argprog5.py --foo1
Namespace(foo1=True, foo2=True)

```


```sh
$ python argprog5.py --foo2
Namespace(foo1=False, foo2=False)

```


```sh
$ python argprog5.py --foo1 --foo2
Namespace(foo1=True, foo2=False)

```


## 6.Type


如果要限定一個argument的 *Type*

例如 *type* 為 *Int*, 可以加入 `type=int` 於程式中

例如以下程式, 我們限定 `type=int` , 表示 `--foo` 只接受整數值



```python argprog6.py 
import argparse
parser = argparse.ArgumentParser( prog='argprog6')
parser.add_argument('--foo', type=int,
                    help='foo of the %(prog)s program, (type: %(type)s)')
args = parser.parse_args()
print args

```


然後輸入 `-h` 看看, help中有顯示出 `type` 了



```sh
$ python argprog6.py -h
usage: argprog6 [-h] [--foo FOO]

optional arguments:
  -h, --help  show this help message and exit
  --foo FOO   foo of the argprog6 program, (type: int)

```


在 `--foo` 後接整數, 是可以的



```sh
$ python argprog6.py --foo 100
Namespace(foo=100)

```


```sh
$ python argprog6.py --foo -100
Namespace(foo=-100)

```


但如果後面接的不是整數, 就會有error



```sh
$ python argprog6.py --foo -1.35
usage: argprog6 [-h] [--foo FOO]
argprog6: error: argument --foo: invalid int value: '-1.35'

```


```sh
$ python argprog6.py --foo abc
usage: argprog6 [-h] [--foo FOO]
argprog6: error: argument --foo: invalid int value: 'abc'

```


## 7.Further Reading


關於 `argparse` 的用法

今天只講了一小部份

若有興趣更深入了解, 請看：

http://docs.python.org/2/library/argparse.html#type
