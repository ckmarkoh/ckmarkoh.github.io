---
layout: post
title: 'Python Standard Library -- logging'
date: 2014-05-08 07:44
comments: true
categories: 
---

## 1. Introduction


在 *python* 程式中想要印出東西, 通常會用 *print* , 要 *print* 東西的時機很多, 例如, *Debug* 時要 *print* , 有 *Error* 時也要 *print* , 正常執行情況下也可能要 *print* 一些資訊


但是當程式寫好後, 原本用 *print* 出來的 *debug message* 就需要手動移除, 諸如此類的問題, 單純用 *pring* 就不方便


如果是用 *logging* , 可以設各種不同的層次, 例如在 *debug* 的時候, 印出 *debug message* , *debug* 結束後就不要印 *debug message*  , 這樣只要調整一個參數即可, 不需要把所有用印出 *debug*  的程式碼都砍掉


*python* 的 *logging* 分成以下五個等級


|等級	| 使用時機 |
|-------|----------------|
|DEBUG	|程式執行時的詳細資訊, 可以找出BUG在哪|
|INFO	  |程式執行時的基本資訊, 用來顯示程式正常執行時的狀況|
|WARNING|程式執行時遇到某些狀況超乎預期, 或未來可能遇到問題, 但程式目前仍可以繼續執行|
|ERROR	|程式執行時發生問題, 可能導致結果產生錯誤|
|CRITICAL|程式執行時遇到嚴重問題, 程式必須馬上停止|

<!--more-->

如果把 *logging* 等級設定在 *debug* , 則 *debug* , *info* 和其他三種等級的訊息都會印出來, 如果等級設定在 *warning* , 則不會印出 *debug* 和 *info* , 以此類推, 因此不需要 *debug message* 時, 只需要把等級設定在 *info* 以及更高等級即可, 因此就不需要砍掉所有印出印出 *debug*  的程式碼


## 2. Implementation


接著是實作部份, 把以下程式碼複製到 *log_debug.py* 這個檔案



```python log_debug.py
import logging
logging.basicConfig(level=logging.DEBUG)
logging.debug('debug message')
logging.info('info message')
logging.warning('warning message')
logging.error('error message')
logging.critical('critical message')

```


其中 `level=logging.DEBUG` 把 *level* 設定在 *debug*

執行這個檔案, *logging* 可以印出下五個等級



```bash
$ python log_debug.py
DEBUG:root:debug message
INFO:root:info message
WARNING:root:warning message
ERROR:root:error message
CRITICAL:root:critical message

```


再來, 把 *log_debug.py* 中的 `level=logging.DEBUG` 改成 `level=logging.WARNING` 

另存新檔為 *log_warning.py* 



```python log_warning.py
import logging
logging.basicConfig(level=logging.WARNING)
logging.debug('debug message')
logging.info('info message')
logging.warning('warning message')
logging.error('error message')
logging.critical('critical message')

```


因為把 *level* 設定在 *warning* , 所以不會印出 *info* 和 *debug* 的 *message*

執行結果如下



```bash
$ python log_warning.py 
WARNING:root:warning message
ERROR:root:error message
CRITICAL:root:critical message

```


## 3. Config Setting



#### 3.1 Logging to a file

前文中提到, 可以從 `basicConfig` 設定 *logging* 的 *level* , 其實 `basicConfig` 也可以做其他設定, 例如把 *log* 輸出到某個檔案, 如下 



```python log_file.py
import logging
logging.basicConfig(filename='log_file.log',level=logging.DEBUG)
logging.debug('debug message')
logging.info('info message')

```


其中, `filename='log_file.log'` 是設定, 要把 *log* 存到哪

到*terminal* 執行完後, *log* 不會直接印出, 



```bash
$ python log_file.py

```


而是存到 *log_file.log* 檔案裡面



```bash
$ cat log_file.log
DEBUG:root:debug message
INFO:root:info message

```


再執行一次看看



```bash
$ python log_file.py

```


第二次執行的 *log* , 不會蓋掉第一次的 *log* , 而是附加在後面



```bash
$ cat log_file.log
DEBUG:root:debug message
INFO:root:info message
DEBUG:root:debug message
INFO:root:info message

```


如果希望不要附加在後面, 而是直接覆寫原本的檔案, 可以加上參數 `filemode='w'` , 如下



```python log_file.py
logging.basicConfig(filename='log_file.log',filemode='w', level=logging.DEBUG)

```


#### 3.2 Changing the format of displayed messages


也可以藉由更改 `basicConfig` 設定, 來更改 *logging* 輸出的格式


例如我們可以把格式改成 *levelname -- message* 的格式, 如下



```python log_format.py
import logging 
logging.basicConfig(format='%(levelname)s -- %(message)s', level=logging.DEBUG)
logging.debug('debug message')
logging.info('info message')

```


執行結果



```bash
$ python log_format.py 
DEBUG -- debug message
INFO -- info message

```


也可以改成印出時間, 如下



``` python log_time.py
import logging
logging.basicConfig(format='%(asctime)s : %(levelname)s : %(message)s', level=logging.DEBUG)
logging.debug('debug message')
logging.info('info message')

```


結果如下



```bash
$ python log_time.py 
2014-05-08 19:11:45,700 : DEBUG : debug message
2014-05-08 19:11:45,701 : INFO : info message

```



## 4. Reference


Python Documentation Logging HOWTO

https://docs.python.org/2/howto/logging.html
