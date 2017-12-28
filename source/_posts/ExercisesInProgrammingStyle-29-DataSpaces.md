---
title: 프로그래밍패턴 - 29.Data Spaces
tags:
  - ExercisesInProgrammingStyle
  - 프로그래밍패턴
  - python
categories:
  - book
date: 2017-09-15 21:32:29
updated: 2017-09-15 21:32:29
layout:
---

* [프로그래밍패턴](http://wikibook.co.kr/programming-patterns/)을 읽고 공부하면서 쓰는 글입니다.

* code는 저자의 github(https://github.com/crista/exercises-in-programming-style) 에 있는 것을 typing만 했습니다. 

## Data Space

* queue를 데이터 공간으로 만들어 두고 thread 활용

* python에서 queue는 therad 환경을 고려하여 만들어졌기 때문에 여러 thread들이 동시에 queue 객체에 데이터를 입력하고 출력하여도 정상적으로 작동하는 것을 보장한다.

```py
import re, sys, operator, queue, threading


word_space = queue.Queue()
freq_space = queue.Queue()

stopwords = set(open('../stop_words.txt').read().split(','))


def process_words():
    word_freqs = {}
    while True:
        try:
            word = word_space.get(timeout=1)
        except queue.Empty:
            break
        if not word in stopwords:
            if word in word_freqs:
                word_freqs[word] += 1
            else:
                word_freqs[word] = 1
    freq_space.put(word_freqs)


for word in re.findall('[a-z]{2,}', open(sys.argv[1]).read().lower()):
    word_space.put(word)


workers = []
for i in range(5):
    workers.append(threading.Thread(target=process_words))

[t.start() for t in workers]

[t.join() for t in workers]


word_freqs = {}
while not freq_space.empty():
    freqs = freq_space.get()
    for k, v in freqs.items():
        if k in word_freqs:
            count = sum(item[k] for item in [freqs, word_freqs])
        else:
            count = freqs[k]
        word_freqs[k] = count


for w, c in sorted(
    word_freqs.items(), key=operator.itemgetter(1), reverse=True)[:25]:
    print(f'{w} - {c}')
```