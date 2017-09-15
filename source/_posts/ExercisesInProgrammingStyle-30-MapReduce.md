---
title: 프로그래밍패턴 - 30.MapReduce
tags:
  - ExercisesInProgrammingStyle
  - 프로그래밍패턴
  - python
categories:
  - book
date: 2017-09-15 21:33:04
updated: 2017-09-15 21:33:04
layout:
---

* [프로그래밍패턴](http://wikibook.co.kr/programming-patterns/)을 읽고 공부하면서 쓰는 글입니다.

* code는 저자의 github(https://github.com/crista/exercises-in-programming-style) 에 있는 것을 typing만 했습니다. 

## MapReduce

* MapReduce 형식은 두 가지 주요 추상화로 이뤄진다.

1. map 함수는 데이터 덩어리와 함수를 인자로 받아 그 함수를 각 덩어리에 독립적으로 적용해 결과 모음을 만들어 낸다.

2. reduce 함수는 결과 모음과 함수를 인자로 취해 그 모음에서 어떤 전역 정보를 추출하기 위해 그 함수를 결과 모음에 적용한다.

* 단어 빈도는 분할 정복법으로 처리할 수 있다.

프로그램 분석
* partition은 generator다.
* map에서 선택된 파일을 200줄씩 잘라서 split_words 함수에 전달한다.
* map은 iterator를 반환한다.
* 그래서 splits는 iterator들로 구성된 리스트다.
* 이제 reduce를 이용하여 count_words 함수에 splits의 element들을 하나씩 전달한다.
* count_words의 반환값과 splits의 element를 순차적으로 처리하는 구조이므로
* splits의 첫 element로 빈 list를 하나 추가한다.

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


def count_words(pairs_list_1, pairs_list_2):
    mapping = dict((k, v) for k, v in pairs_list_1)
    for p in pairs_list_2:
        if p[0] in mapping:
            mapping[p[0]] += p[1]
        else:
            mapping[p[0]] = 1
    return mapping.items()


def read_file(path_to_file):
    with open(path_to_file) as f:
        data = f.read()
    return data


def sort(word_freq):
    return sorted(
        word_freq,
        key=operator.itemgetter(1),
        reverse=True)


splits = list(
    map(split_words, partition(read_file(sys.argv[1]), 200)))
splits.insert(0, [])
word_freqs = sort(reduce(count_words, splits))


for (w, c) in word_freqs[0:25]:
    print(f'{w} - {c}')

```