---
title: 프로그래밍패턴 - 03.Monolith
tags:
  - ExercisesInProgrammingStyle
  - 프로그래밍패턴
  - python
categories:
  - book
date: 2017-09-19 20:29:17
updated: 2017-09-19 20:29:17
layout:
---

* [프로그래밍패턴](http://wikibook.co.kr/programming-patterns/)을 읽고 공부하면서 쓰는 글입니다.

* code는 저자의 github(https://github.com/crista/exercises-in-programming-style) 에 있는 것을 typing만 했습니다. 

## Monolith

* 말 그대로 일체형

* 요즘도 시간에 쫓기면서 코드를 짜다보면 이런 일이 종종 생긴다.

* 내공부족. 코드를 보면 답답하다.

```py
"""
순환 복잡도(cyclomatic complexity)는 프로그램 본문의 복잡도, 특히 제어 흐름 경로의 양을 측정하기 위해 개발된 측정 지표다.
경험적으로 프로그램 본문 한 조각의 순환 복잡도가 높을수록 프로그램을 이해하기도 어려워진다.
순환 복잡도 계산은 프로그램 본문을 방향 그래프로 생각하고 다음 계산식에 따른다.
CC = E - N + 2P
E(Edge): 변의 수
N: 노드 수
P: 출구 노드 수
"""
import sys, string


word_freqs = []

with open('../stop_words.txt') as f:
    stop_words = f.read().split(',')
    """
    stop_words = f.read()
    단순히 f.read()로 처리하면 하나의 파일을 읽어서 하나의 문자열로 만든다.
    print(type(stop_words)) # <class 'str'>
    이 문자열은 ,로 구분된 단어들이다. 그래서 split(',')을 하면
    단어가 element인 list가 된다.
    """

stop_words.extend(list(string.ascii_lowercase))
"""
string.ascii_lowercase는 a부터 z까지 소문자로 된 문자열을 반환한다.
print(string.ascii_lowercase)
따라서 위 코드를 실행하면 stop_words list와 소문자 알파벳 하나하나가 
element인 list가 합쳐진다.
"""

for line in open(sys.argv[1]):
    start_char = None
    i = 0
    for c in line:
        if start_char == None:
            if c.isalnum():
                start_char = i
        else:
            if not c.isalnum():
                found = False
                word = line[start_char:i].lower()
                if word not in stop_words:
                    pair_index = 0
                    for pair in word_freqs:
                        if word == pair[0]:
                            pair[1] += 1
                            found = True
                            found_at = pair_index
                            break
                        pair_index += 1
                    if not found:
                        word_freqs.append([word, 1])
                    elif len(word_freqs) > 1:
                        for n in reversed(range(pair_index)):
                            if word_freqs[pair_index][1] > word_freqs[n][1]:
                                word_freqs[n], word_freqs[pair_index] = word_freqs[pair_index], word_freqs[n]
                                pair_index = n
                start_char = None
        i += 1

for tf in word_freqs[0:25]:
    print(f'{tf[0]} - {tf[1]}')

```