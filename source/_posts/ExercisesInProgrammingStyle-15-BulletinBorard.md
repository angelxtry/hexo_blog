---
title: 프로그래밍패턴 - 15.Bulletin Board
tags:
  - ExercisesInProgrammingStyle
  - 프로그래밍패턴
  - python
categories:
  - book
date: 2017-09-07 18:47:56
updated: 2017-09-07 18:47:56
layout:
---

* [프로그래밍패턴](http://wikibook.co.kr/programming-patterns/)을 읽고 공부하면서 쓰는 글입니다.

* code는 저자의 github(https://github.com/crista/exercises-in-programming-style) 에 있는 것을 typing만 했습니다. 

## Bulletin Board

* 규모가 큰 문제를 어떤 추상 형태(객체, 모듈 또는 이와 비슷한)를 이용해
개체로 분해한다.

* 어떤 행동을 하기 위해 개체를 절대로 직접 호출하지 않는다.

* 이벤트
  - 특정 시점에 한 구성 요소가 만들어 내고, 이를 기다리는 다른 구성 요소에 배포하기로 되어있는 자료 구조

* 발행
  - 이벤트 배포 기반 구조에서 제공하는 연산의 하나로, 구성 요소가 다른 구성 요소에 이벤트를 배포할 수 있게 한다.

* 구독
  - 이벤트 배포 기반 구조에서 제공하는 연산의 하나로, 구성 요소가 특정 이벤트에 관심을 표시할 수 있게 한다.

* 일반적으로 발행-구독 구조는 이벤트 타입을 구독 해지하는 개념도 지원하다.

* EventManager에서 unsubscribe 연산을 지원하게 한다.

```py
import sys, re, operator, string


class EventManager:
    def __init__(self):
        self._subscriptions = {}

    def subscribe(self, event_type, handler):
        if event_type in self._subscriptions:
            self._subscriptions[event_type].append(handler)
        else:
            self._subscriptions[event_type] = [handler]

    def publish(self, event):
        event_type = event[0]
        if event_type in self._subscriptions:
            for h in self._subscriptions[event_type]:
                h(event)


class DataStorgae:
    def __init__(self, event_manager):
        self._event_manager = event_manager
        self._event_manager.subscribe('load', self.load)
        self._event_manager.subscribe('start', self.produce_words)

    def load(self, event):
        path_to_file = event[1]
        with open(path_to_file) as f:
            self._data = f.read()
        pattern = re.compile('[\W_]+')
        self._data = pattern.sub(' ', self.data).lower()

    def produce_words(self, event):
        data_str = ''.join(self._data)
        for w in data_str.split():
            self._event_manager.publish(('word', w))
        self._event_manager.publish(('eof', None))


class StopWordFilter:
    def __init__(self, event_manager):
        self._stop_words = []
        self._event_manager = event_manager
        self._event_manager.subscribe('load', self.load)
        self._event_manager.subscribe('word', self.is_stop_word)

    def load(self, event):
        with open('../stop_word.txt') as f:
            self._stop_words = f.read().split(',')
        self._stop_words.extend(list(string.ascii_lowercase))

    def is_stop_word(self, event):
        word = event[1]
        if word not in self._stop_words:
            self._event_manager.publish(('valid_word', word))


class WordFrequencyCounter:
    def __init__(self, event_manager):
        self._word_freqs = {}
        self._event_manager = event_manager
        self._event_manager.subscribe('valid_word', self.increment_count)
        self._event_manager.subscribe('print', self.print_freqs)

    def increment_count(self, event):
        word = event[1]
        if word in self._word_freqs:
            self._word_freqs[word] += 1
        else:
            self._word_freqs[word] = 1

    def print_freqs(self, event):
        word_freqs = sorted(
            self._word_freqs.items(),
            operator.itemgetter(1),
            reverse=True)
        for (w, c) in word_freqs[0:25]:
            print(f'{w} - {c}')


class WordFrequencyApplication:
    def __init__(self, event_manager):
        self._event_manager = event_manager
        self._event_manager.subscribe('run', self.run)
        self._event_manager.subscribe('eof', self.stop)

    def run(self, event):
        path_to_file = event[1]
        self._event_manager.publish(('load', path_to_file))
        self._event_manager.publish(('start', None))

    def stop(self, event):
        self._event_manager.publish(('print', None))


em = EventManager()
DataStorgae(em)
StopWordFilter(em)
WordFrequencyCounter(em)
WordFrequencyApplication(em)
em.publish('run', sys.argv[1])

```