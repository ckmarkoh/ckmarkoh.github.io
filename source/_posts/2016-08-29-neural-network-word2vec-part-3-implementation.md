---
layout: post
title: 'word2vec (part 3 : Implementation)'
date: 2016-08-29 11:17
comments: true
categories: ['Natural Language Processing','Neural Networks']
---

## Introduction


本文接續 [word2vec (part2)](/blog/2016/07/12/-word2vec-neural-networks-part-2-backward-propagation) ，介紹如何根據推導出來的 *backward propagation* 公式，從頭到尾實作一個簡易版的 *word2vec* 。

本例的 input layer 採用 *skip-gram* ， output layer 採用 *negative sampling*


本例用唐詩語料庫：https://github.com/ckmarkoh/coscup_nndl/blob/master/poem.txt


首先，載入所需的模組



```python
import json
from collections import Counter, OrderedDict
import numpy as np
import random
import math

```

<!--more-->


## Build Dictionray


再來是建立字典，即將每個字給一個id來對應。



```python
def LearnVocabFromTrainFile():
		
    # 開啟唐詩語料庫
    f = open("poem.txt")
 
    # 統計唐詩語料庫中每個字出現的頻率
    vcount = Counter()
    for line in f.readlines():
        for w in line.decode("utf-8").strip().split():
            vcount.update(w)
            
    # 僅保留出現次數大於五的字，並按照出現次數排序
    vcount_list = sorted(filter(lambda x: x[1] >= 5, vcount.items())
                         , reverse=True, key=lambda x: x[1])
                         
    # 建立字典，將每個字給一個id ，字為 key, id 為 value
    vocab_dict = OrderedDict(map(lambda x: (x[1][0], x[0]), enumerate(vcount_list)))
    
    # 建立詞頻統計用的字典，給定某字，可查到其出現頻率
    vocab_freq_dict = OrderedDict(map(lambda x: (x[0], x[1]), vcount_list))
    return vocab_dict, vocab_freq_dict
    
vocab_dict, vocab_freq_dict =  LearnVocabFromTrainFile()


```


印出字典檔，每個字對應到一個id（編號）



```python
for w,wid in vocab_dict.items():
    print "%s : %s"%(w,wid)

```


結果如下：



```sh
不 : 0
人 : 1
山 : 2
無 : 3
風 : 4
......
謏 : 5496
笮 : 5497
躠 : 5498
噆 : 5499

```


印出詞頻統計用的字典，給定某字，可查詢到其出現頻率：



```python
for w,wfreq in vocab_freq_dict.items():
    print "%s : %s"%(w,wfreq)

```


結果如下：



```sh
不 : 26426
人 : 20966
山 : 16056
無 : 15795
風 : 15618
...
謏 : 5
笮 : 5
躠 : 5
噆 : 5

```


## Build Unigram Table


本例採用 *negative sampling* ，需要先建立 *unigram table* 以便進行 *negative sampling* 。


所謂的 *Unigram Table* 即是一個 *array* ，其中每個元素為某字的id，而某字的頻率，即為此id在此 *table* 中出現的次數的 0.75次方。


例如，id 為 5496 的字，詞頻為 5 ，則在此 *Unigram Table* 中，5496 的次數為：


$$

5^{0.75} = 3.34 \approx 3

$$


由於 *array* 中的元素個數必須是整數，所以 5496 在 *Unigram Table* 中出現三次。


建立 *Unigram Table* 的程式碼如下：



```python
def InitUnigramTable(vocab_freq_dict):
    table_freq_list = map(lambda x: (x[0], int(x[1][1] ** 0.75)), enumerate(vocab_freq_dict.items()))
    table_size = sum([x[1] for x in table_freq_list])
    table = np.zeros(table_size).astype(int)
    offset = 0
    for item in table_freq_list:
        table[offset:offset + item[1]] = item[0]
        offset += item[1]

    return table
    
table = InitUnigramTable(vocab_freq_dict)

```


得出的 *Unigram Table* 如下：



```
[   0    0    0    0    0    0    0    0    0    0    0    0    0    0    0    
0    0    0    0  ... , 5495 5495 5495 5496  5496 5496 5497 5497 5497 5498 
5498 5498 5499 5499 5499]

```


## Training word2vec



```python
def train(vocab_dict, vocab_freq_dict, table):
		
    total_words = sum([x[1] for x in vocab_freq_dict.items()])
    vocab_size = len(vocab_dict)

    # 參數設定
    layer1_size = 30 # hidden layer 的大小，即向量大小
    window = 2 # 上下文寬度的上限
    alpha_init = 0.025 # learning rate
    sample = 0.001 # 用來隨機丟棄高頻字用
    negative = 10 # negative sampling 的數量
    ite = 2 # iteration 次數
    
    # Weights 初始化
    # syn0 : input layer 到 hidden layer 之間的 weights ，用隨機值初始化
    # syn1 : hidden layer 到 output layer 之間的 weights ，用0初始化
    syn0 = (0.5 - np.random.rand(vocab_size, layer1_size)) / layer1_size 
    syn1 = np.zeros((layer1_size, vocab_size))
    
    # 印出進度用
    train_words = 0 # 總共訓練了幾個字
    p_count = 0
    avg_err = 0.
    err_count = 0
    
    for local_iter in range(ite):
        print "local_iter", local_iter
        f = open("poem.txt")
        for line in f.readlines():
            
            #用來暫存要訓練的字，一次訓練一個句子
            sen = []
            
            # 取出要被訓練的字
            for word_raw in line.decode("utf-8").strip().split():
                last_word = vocab_dict.get(word_raw, -1)
                
                # 丟棄字典中沒有的字（頻率太低）
                if last_word == -1:
                    continue
                cn = vocab_freq_dict.get(word_raw)
                ran = (math.sqrt(cn / float(sample * total_words + 1))) * (sample * total_words) / cn
                
                # 根據字的頻率，隨機丟棄，頻率越高的字，越有機會被丟棄
                if ran < random.random():
                    continue
                train_words += 1
                
                # 將要被訓練的字加到 sen
                sen.append(last_word)
                
            # 根據訓練過的字數，調整 learning rate
            alpha = alpha_init * (1 - train_words / float(ite * total_words + 1))
            if alpha < alpha_init * 0.0001:
                alpha = alpha_init * 0.0001
                
            # 逐一訓練 sen 中的字
            for a, word in enumerate(sen):
            
            		# 隨機調整 window 大小
                b = random.randint(1, window)
                for c in range(a - b, a + b + 1):
                    
                    # input 為 window 範圍中，上下文的某一字
                    if c < 0 or c == a or c >= len(sen):
                        continue
                    last_word = sen[c]
										
                    # h_err 暫存 hidden layer 的 error 用
                    h_err = np.zeros((layer1_size))
                    
                    # 進行 negative sampling
                    for negcount in range(negative):
                    
                    		# positive example，從 sen 中取得，模型要輸出 1
                        if negcount == 0:
                            target_word = word
                            label = 1
                        
                        # negative example，從 table 中抽樣，模型要輸出 0 
                        else:
                            while True:
                                target_word = table[random.randint(0, len(table) - 1)]
                                if target_word not in sen:
                                    break
                            label = 0
                        
                        # 模型預測結果
                        o_pred = 1 / (1 + np.exp(- np.dot(syn0[last_word, :], syn1[:, target_word])))
                        
                        # 預測結果和標準答案的差距
                        o_err = o_pred - label
                        
                        # backward propagation
                        # 此部分請參照 word2vec part2 的公式推導結果
                        
                        # 1.將 error 傳遞到 hidden layer                        
                        h_err += o_err * syn1[:, target_word]
                        
                        # 2.更新 syn1
                        syn1[:, target_word] -= alpha * o_err * syn0[last_word]
                        avg_err += abs(o_err)
                        err_count += 1
                    
                    # 3.更新 syn0
                    syn0[last_word, :] -= alpha * h_err
                    
                    # 印出目前結果
                    p_count += 1
                    if p_count % 10000 == 0:
                        print "Iter: %s, Alpha %s, Train Words %s, Average Error: %s" \
                              % (local_iter, alpha, 100 * train_words, avg_err / float(err_count))
                        avg_err = 0.
                        err_count == 0.
                        
        # 每一個 iteration 儲存一次訓練完的模型
        model_name = "w2v_model_blog_%s.json" % (local_iter)
        print "save model: %s" % (model_name)
        fm = open(model_name, "w")
        fm.write(json.dumps(syn0.tolist(), indent=4))
        fm.close()

```


開始訓練：



```python
train(vocab_dict, vocab_freq_dict, table)

```


輸出結果如下，可以看到，當訓練過的字數增加時， Error 也跟著降低

大概要花幾十分鐘左右訓練完



```sh
Iter: 0, Alpha 0.0249923666923, Train Words 475200, Average Error: 0.499999254842
Iter: 0, Alpha 0.0249846739501, Train Words 954100, Average Error: 0.249998343836
Iter: 0, Alpha 0.0249771900316, Train Words 1420000, Average Error: 0.166660116256
Iter: 0, Alpha 0.0249693430813, Train Words 1908500, Average Error: 0.124949913475
Iter: 0, Alpha 0.024961329072, Train Words 2407400, Average Error: 0.0993522008349
Iter: 0, Alpha 0.0249531817368, Train Words 2914600, Average Error: 0.0787704454331
Iter: 0, Alpha 0.0249453540624, Train Words 3401900, Average Error: 0.06351951221
Iter: 0, Alpha 0.0249377801891, Train Words 3873400, Average Error: 0.0495117808015
..........

```


## Show Result


檢視 word2vec 訓練結果的方法，即是看使用 *cosine similarity* 計算，是否能得出與某字語意相近的字。



```python

# 讀取訓練好的模型
f2 = open("w2v_model_1.json", "r") 
w2v_model = np.array(json.loads("".join(f2.readlines())))
f2.close()

vocab_dict_reversed = OrderedDict([(x[1], x[0]) for x in vocab_dict.items()])

# 計算 cosine similarity 最高的前五字
def get_top(word):
    wid = vocab_dict.get(word)
    
    # 將某字與模型中所有的字向量做內積
    dot_result = np.dot(w2v_model, np.expand_dims(w2v_model[wid], axis=1))
    norm = np.sqrt(np.sum(np.power(w2v_model.T, 2), axis=0))
    
    # 計算 cosine similarity
    cosine_result = np.divide(dot_result[:, 0], norm*norm[wid])
    
    # 根據 cosine similarity 的值排序
    final_result = sorted(filter(lambda x:x[0] != wid, 
                          [(x[0], x[1]) for x in enumerate(cosine_result)]),
                          key=lambda x: x[1], reverse=True)
    print word
    
    # 印出語意最接近的前五字，以及其 cosine similarity
    for x in final_result[:5]:
        print vocab_dict_reversed.get(x[0]), x[1]

```


分別計算「山、峰、河、日」這四字語意最相近的字



```python
get_top(u"山")
get_top(u"峰")
get_top(u"河")
get_top(u"日")

```


結果如下，可看出，計算所得出語意最相近的字，實際上，語意也相近，例如，山和峰、嶺的語意都很接近。



```sh
山
嶺 0.854901128361
嵩 0.846620438864
峰 0.842831270385
岡 0.838129842909
嶂 0.834701215189
峰
山 0.842831270385
嶽 0.83917452917
嶺 0.8219837161
頂 0.821088331571
嶂 0.809565794884
河
湟 0.787726187693
涇 0.770652269018
淮 0.751135710239
川 0.742243126005
汾 0.740643816278
日
旦 0.869047480855
又 0.842383624714
曛 0.830549707539
夕 0.826327222048
暉 0.82616774597

```


向量加減運算後的 *cosine similarity* ，例如： 女 + 父 - 男 = 母



```python
def get_calculated_top(w1, w2, w3):
    wid1, wid2, wid3 = vocab_dict.get(w1), vocab_dict.get(w2), vocab_dict.get(w3)
    v1, v2, v3 = w2v_model[wid1], w2v_model[wid2], w2v_model[wid3]
    
    # 得出加減運算後的向量
    combined_vec = v1 + (v2 - v3)
    dot_result = np.dot(w2v_model, np.expand_dims(combined_vec, axis=1))
    norm = np.sqrt(np.sum(np.power(w2v_model.T, 2), axis=0))
    cvec_norm = np.sqrt(np.sum(np.power(combined_vec, 2)))
    cosine_result = np.divide(dot_result[:, 0], norm * cvec_norm)

    final_result = sorted(filter(lambda x: x[0] not in [wid1, wid2, wid3],
                                 [(x[0], x[1]) for x in enumerate(cosine_result)]),
                          key=lambda x: x[1], reverse=True)
    print "%s + %s - %s" % (w1, w2, w3)
    for x in final_result[:5]:
        print vocab_dict_reversed.get(x[0]), x[1]

```



```python
get_calculated_top(u"女", u"父", u"男")

```


結果如下，如預期，運算結果的語意接近「母」：



```sh
女 + 父 - 男
母 0.731002049447
娥 0.707469857054
客 0.69027387716
娃 0.687831493041
侶 0.681667240226

```
