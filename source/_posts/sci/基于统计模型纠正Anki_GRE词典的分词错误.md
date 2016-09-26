---
title: 基于统计模型纠正Anki_GRE词典的分词错误
date: 2016-09-24 21:15:35
categories: science
tags: Anki, 统计, 马尔科夫模型, 分词
---

在Anki软件中，使用的极品GRE卡牌组中的单词解释不知道怎么回事，单词经常被莫名其妙的断开。

比如`limp and flabby`被写成了`li mp and flabb y`，读起来很不爽。于是想通过程序自动纠正这种错误。

## 方案1
一开始想到了一个很naive的方法：首先建立一个词典，然后每次读入一个词（空格分开）判断一下该词是否存在于词典中，如果不存在，判断该词与下一个词的组合是否在词典中，如果在，就把这两个词组合在一起并保存。

比如`flabb`不存在于词典，而`flabby`存在于词典，这样就能把这个错误纠正。

但是也有很多情况，即使一个单词被错误的分成了两个部分，但这两个部分依然是正确的单词。比如`li`和`mp`都存在于词典中，这样的方法就不能纠正这样的错误。

这是因为这个方案实在是too naive了，完全没有利用到上下文的关系。

## 方案2
使用google的API，google会提供搜索建议，比如搜索`li mp and flabb y`，google会返回提示你是否要查询的是`limp and flabby`。

google搜索虽然功能很强大，但是由于众所周知的原因，最后没有采用。

## 方案3
参考了`数学之美`和`Beautiful Data`中的分词方法，实际上就是比较$P(a_1,a_2,...,a_n)$与$P(b_1,b_2,...,b_n)$，其中$a_i,b_i$为两种分词方法第$i$个单词。

使用2阶模型，即马尔科夫模型，$P(x_1,x_2,...x_i)=P(x_1|x_2)*P(x_2|x_3)...*P(x_{n-1}|x_n)$。使用了`Beautiful Data`提供的统计数据。每次尝试合并句子中的一个空格，并选出概率较大的句子作为结果。注意原始分词方案增加了权重，这是因为原始分词方案大概率是正确的，因此给它加上一个权重。最后的正确率虽然达不到100%，但还是比较令人满意了。

完整代码如下，
```python

import sqlite3
import re
import urllib
import requests
import json
import time
import wordsegment
from math import log10

# words dictionary
words=frozenset()
with open("words.txt") as f:
    words=frozenset([line.strip() for line in f])

# correct with google engine
def correct_google(s):
    def extract_tag(s):
        flag=False
        ret=[]
        tmp=""
        for c in s:
            if flag and c=="<":
                ret.append(tmp)
                tmp=""
                flag=False
            if flag:
                tmp+=c
            if not flag and c==">":
                flag=True
        return ret

    url="http://www.google.co.jp/complete/search?client=serp&hl=zh-CN&gs_rn=64&gs_ri=serp&cp=13&gs_id=3ms&q=%s&xhr=t"
    url=url%urllib.quote(s)

    while True:
        try:
            req=requests.get(url, timeout=10)

            js=json.loads(req.text)
            flag=False
            if "o" in js[2]:
                s1=js[2]["p"]
                s2=js[2]["o"]
                t1=extract_tag(s1)
                t2=extract_tag(s2)
                for i in range(len(t1)):
                    # correct word
                    if(t1[i].replace(" ","")==t2[i]):
                        print t1[i], t2[i]
                        s=s.replace(t1[i], t2[i])
                        flag=True
            break
        except Exception, ex:
            print str(ex)
            time.sleep(1)

    return flag,s

# correct using dictionary
def correct_dict(s):
    prev=""
    w_list=s.split(" ")[0:-1]
    rst=[]
    flag=False
    for w in w_list:
        if not w.lower() in words and (prev+w).lower() in words:
            del rst[-1]
            rst.append(prev+w)
            flag=True
        else:
            rst.append(w)
        prev=w

    return flag, ' '.join(rst)

# correct using segmentation and word score
def correct_segmentation(s):

    # generate all possible sentences
    def generate_sentence(words):
        # original sentence
        yield words, 0

        # sentence with one space removed
        for i in range(1,len(words)):
            tmp=words[:]
            tmp[i-1:i+1]=[tmp[i-1]+tmp[i]]
            yield tmp, i

    def score_sentence((words, pos)):
        words=['<s>',]+words

        # add weight for original sentence
        ret=5 if pos==0 else 0
        prev="<s>"
        for i in range(1,len(words)):
            ret+=log10(wordsegment.score(words[i],prev))
            prev=words[i]
        return ret, pos

    lower_s = s.lower()
    words = lower_s.split(' ')

    max_s, pos=max(generate_sentence(words),key=score_sentence)
    max_s=" ".join(max_s)

    if pos!=0:
        ret=s.split(' ')
        ret[pos-1:pos+1]=[ret[pos-1]+ret[pos]]
        ret=' '.join(ret)
        print ret
        return True, ret

    return False, s


words_re=re.compile('(\w+\s+)+\w+')

conn=sqlite3.connect('d:\xxx\Documents\Anki\User 1\collection.anki2')
for row in conn.execute('select id, flds from notes'):
    idd, fld = row
    s_list=[m.group(0) for m in words_re.finditer(fld)]
    flag = False
    for s in s_list:
        # corr, corr_s=correct_dict(s)
        corr, corr_s=correct_segmentation(s)

        if corr:
            flag=True
            fld=fld.replace(s,corr_s)
    if flag:
        conn.execute('update notes set flds=? where id=?', (fld, idd))
        conn.commit()


conn.close()
```