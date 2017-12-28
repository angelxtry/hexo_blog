---
title: 프로그래밍패턴 - 31.Double MapReduce
tags:
  - ExercisesInProgrammingStyle
  - 프로그래밍패턴
  - python
categories:
  - book
date: 2017-09-15 21:33:34
updated: 2017-09-15 21:33:34
layout:
---

* [프로그래밍패턴](http://wikibook.co.kr/programming-patterns/)을 읽고 공부하면서 쓰는 글입니다.

* code는 저자의 github(https://github.com/crista/exercises-in-programming-style) 에 있는 것을 typing만 했습니다. 

## Double MapReduce

* 주목할 만한 주요한 내용은 재편성 후 단어 세기를 병렬로 할 수 있다는 점인데, 단어 당 데이터를 재편성했기 때문이다.
* 그러므로 두 번째 맵 함수를 단어 세기에 적용할 수 있다.

* 이러한 재편성을 통해 파일 내 유일한 단어당 단어 세기 스레드/프로세서를 하나씩 사용할 수 있다.

* 어떤 자료 구조는 병렬로 처리하기가 쉽지 않지만 흔히 해당 데이터를 병렬화할 수 있게 하는 변형이 존재한다.

* 복잡한 데이터 집약적인 문제를 병렬화할 때는 재편성을 여러 번 거쳐야 할 수 있다.
 
* 데이터 센터 규모에서는 병렬화가 단순한 작어르 수행하는 서버에 데이터 구역을 보내는 것으로 끝난다.
 
프로그램 분석
* splits는 tuple로 구성된 list인 element로 구성된 list다.
* 이 list를 regroup에 전달한다.
* regroup에서는 데이터를 dict를 이용하여 재정렬한다.
* dict의 key는 모든 단어이고 value는 key와 동일한 단어로 구성된 tuple이다.
* 정렬된 dict를 map을 이용하여 한쌍의 key, value씩 순차적으로 count_words에 전달한다.
* count_words 함수에서 mapping[0]은 전달받은 key, 즉 단어다.
* mapping[1]은 동일한 단어의 tuple로 구성된 list다.
* 이 list를 순회하면서 두 번째 인자인 1을 모두 더한다. 이것이 빈도 수가 된다.

```py
import sys, re, operator, string
from functools import reduce


def partition(data_str, nlines):
    lines = data_str.split('\n')
    for i in range(0, len(lines), nlines):
        yield '\n'.join(lines[i:i+nlines])


def split_words(data_str):
    def _scan(str_data):
        pattern = re.compile('[\W_]+')
        return pattern.sub(' ', str_data).lower().split()

    def _remove_stop_words(word_list):
        with open('../stop_words.txt') as f:
            stop_words = f.read().split(',')
        stop_words.extend(list(string.ascii_lowercase))
        return [w for w in word_list if not w in stop_words]
    
    result = []
    words = _remove_stop_words(_scan(data_str))
    for w in words:
        result.append((w, 1))
    return result


def regroup(pairs_list):
    mapping = {}
    for pairs in pairs_list:
        for p in pairs:
            if p[0] in mapping:
                mapping[p[0]].append(p)
            else:
                mapping[p[0]] = [p]
    return mapping


def count_words(mapping):
    def add(x, y):
        return x+y
    return (mapping[0], reduce(add, (pair[1] for pair in mapping[1])))


def read_file(path_to_file):
    with open(path_to_file) as f:
        data = f.read()
    return data


def sort(word_freq):
    return sorted(word_freq, key=operator.itemgetter(1), reverse=True)


splits = list(map(split_words, partition(read_file(sys.argv[1]), 200)))
splits_per_word = regroup(splits)
word_freqs = sort(map(count_words, splits_per_word.items()))


for (w, c) in word_freqs[0:25]:
    print(f'{w} - {c}')

```