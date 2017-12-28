---
title: 프로그래밍패턴 - 04.Cook Book
tags:
  - ExercisesInProgrammingStyle
  - 프로그래밍패턴
  - python
categories:
  - book
date: 2017-09-19 20:33:52
updated: 2017-09-19 20:33:52
layout:
---

* [프로그래밍패턴](http://wikibook.co.kr/programming-patterns/)을 읽고 공부하면서 쓰는 글입니다.

* code는 저자의 github(https://github.com/crista/exercises-in-programming-style) 에 있는 것을 typing만 했습니다. 


## Cook Book

* 절차적 추상화를 이용해 규모가 큰 문자렐 작은 단위로 분할함으로써 제어 흐름 복잡도를 완화한다.

* 프로시저는 상태를 전역 변수 형태로 공유

* 요리법에 따라 재료의 상태를 변경하는 것과 마찬가지로 공유 변수의 상태를 변경한다.

```py
import sys, string

data = []
words = []
word_freqs = []


"""
파일을 읽어서 모든 데이터를 list(f.read())를 이용하여 data에 저장
data에는 list형식으로 모든 element가 char 1개씩 저장된다.
Question: 왜 단어 단위로 저장하지 않고 문자단위로 저장했을까?
"""
def read_file(path_to_file):
    global data
    with open(path_to_file) as f:
        data = data + list(f.read())


"""
알파벳이나 숫자가 아닌 문자를 공백으로 변경한다.
모든 알파벳은 소문자로 변경한다.
"""
def filter_chars_and_normalize():
    global data
    for i in range(len(data)):
        if not data[i].isalnum():
            data[i] = ' '
        else:
            data[i] = data[i].lower()


"""
문자가 element로 저장되어있던 list를 하나의 문자열로 변환
하나의 문자열을 공백문자를 기준으로 분리하여 단어 element로 구성된 리스트로 변환
"""
def scan():
    global data
    global words
    data_str = ''.join(data)
    words = words + data_str.split()


"""
words에서 stop_words가 포함된 index를 확인하여 indexes 리스트를 작성
words 리스트에서 뒤에서부터 해당 index를 삭제한다.
앞에서부터 삭제하면 index가 흐트러진다.
"""
def remove_stop_words():
    global words
    with open('../stop_words.txt') as f:
        stop_words = f.read().split(',')
    
    stop_words.extend(list(string.ascii_lowercase))
    indexes = []
    for i in range(len(words)):
        if words[i] in stop_words:
            indexes.append(i)
    for i in reversed(indexes):
        words.pop(i)


"""
python -m cProfile 04-cookbook.py ../pride-and-prejudice.txt
프로파일링해보면 이 함수에서 가장 시간이 많이 소요된다.
keys = [wd[0] for wd in word_freqs]
특히 이 라인이 매번 리스트를 생성하기 때문에 오버헤드가 된다.
"""
def frequencies():
    global words
    global word_freqs
    for w in words:
        keys = [wd[0] for wd in word_freqs]
        if w in keys:
            word_freqs[keys.index(w)][1] += 1
        else:
            word_freqs.append([w, 1])


def sort():
    global word_freqs
    # word_freqs.sort(lambda x, y: cmp(y[1], x[1]))
    word_freqs.sort(key=lambda x: x[1], reverse=True)


read_file(sys.argv[1])
filter_chars_and_normalize()
scan()
remove_stop_words()
frequencies()
sort()

for tf in word_freqs[0:25]:
    print(f'{tf[0]} - {tf[1]}')
```