---
title: 프로그래밍패턴 - 18.Aspects
tags:
  - ExercisesInProgrammingStyle
  - 프로그래밍패턴
  - python
categories:
  - book
date: 2017-09-08 07:49:57
updated: 2017-09-08 07:49:57
layout:
---

* [프로그래밍패턴](http://wikibook.co.kr/programming-patterns/)을 읽고 공부하면서 쓰는 글입니다.

* code는 저자의 github(https://github.com/crista/exercises-in-programming-style) 에 있는 것을 typing만 했습니다. 

## Aspects

* 이 형식은 기존 프로그램에서 지정된 위치 전후로 임의의 코드를 삽입하는 구체적인 목적을 위한 제한된 반영(restrained reflection) 으로 설명할 수 있다.

* 이렇게 하는 이유 중 하나는 소스코드에 대한 접근이나 변경은 원하지 않지만 해당 프로그램의 함수에 기능을 추가하고 싶은 경우나 일반적으로 프로그램 전체에 흩어져 있는 코드를 지역화해 개발을 단순화하는 것이다.

* 이 형식의 제약조건 중 하나는 영향을 받는 함수나 그 함수의 호출 위치를 편집하지 않고 부가 기능을 도입해야 한다는 것이다.

* 프로파일 할 함수 각각에 대해 기호표에 있는 그 이름의 결합 내용을 해당 함수를 감싼 함수로 대체한다.

* 예를 들면, extract_words라는 이름과 함수 몸체와의 결합을 끊고 extract_words라는 이름을 extract_words 함수를 매개변수로 취하는 profile 함수의 인스턴스와 결합시킨다.


```py
import sys, re, operator, string, time


def extract_words(path_to_file):
    with open(path_to_file) as f:
        str_data = f.read()
    pattern = re.compile('[\W_]+')
    word_list = pattern.sub(' ', str_data).lower().split()
    with open('../stop_words.txt') as f:
        stop_wrods = f.read().split(',')
    stop_wrods.extend(list(string.ascii_lowercase))
    return [w for w in word_list if not w in stop_wrods]


def frequescies(word_list):
    word_freqs = {}
    for w in word_list:
        if w in word_freqs:
            word_freqs[w] += 1
        else:
            word_freqs[w] = 1
    return word_freqs


def sort(word_freqs):
    return sorted(word_freqs.items(), key=operator.itemgetter(1), reverse=True)


def profile(f):
    def profilewrapper(*arg, **kw):
        start_time = time.time()
        ret_value = f(*arg, **kw)
        elapsed = time.time() - start_time
        print(f'{f.__name__}(...) took {elapsed} secs')
        return ret_value
    return profilewrapper


tracked_functions = [extract_words, frequescies, sort]


for func in tracked_functions:
    globals()[func.__name__] = profile(func)


word_freqs = sort(frequescies(extract_words(sys.argv[1])))


for (w, c) in word_freqs[0:25]:
    print(f'{w} - {c}')

```