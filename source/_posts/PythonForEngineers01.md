---
title: 엔지니어를 위한 파이썬 11. 프로그램 최적화
tags:
  - 최적화
  - python
categories:
  - book
date: 2018-02-06 21:54:17
updated: 2018-02-06 21:54:17
---

* 파이썬 격월세미나에서 받은 엔지니어를 위한 파이썬을 읽고 있다.
* 재밌다. good.
* ipython을 활용하는 법이 많이 포함되어있다.
* jupyter notebook에서도 그대로 되는지 모르겠다.
* 책 순서와 상관없이 필요한 부분을 정리해본다.

## 최적화를 위한 4가지 접근법
1. 벙목해소
    * 코딩 방법에 따른 최적화
    * 메모리의 효율적인 사용
    * 프로파일러 활용
2. 병렬처리
    * SIMD 사용
    * 멀티스레드 적용
    * 멀티프로세스 적용
3. 고속 라이브러리(타 언어 구현) 활용
    * Cython 사용
    * C/C++ 라이브러리 활용(ctypes)
4. JIT 컴파일러 활용
    * Numba
    * Numexpr

## 병목해소
* 프로그램의 전체 실행 속도는 가장 처리가 오래 걸리는 부분에 의해 제한 받는다.
* 단순히 계산이 오래 걸리는 부분을 찾아 개선하는 것뿐만 아니라 CPU, 메모리, 하드디스크 간에 데이터를 주고받을 때 발생하는 지연 시간을 고려하는 것이 중요하다.

### 1. 코딩 방법에 따른 최적화
* 선입견 없이 여러가지 방식을 시도해본다.
* 파이썬의 내장 함수 및 표준 라이브러리를 최대한 활용한다.
* 반복문(for, while) 사용을 최대한 피한다.

1. 선입견 없이 여러가지 방식을 시도해본다.
```py
In [13]: %timeit x = 1213; x = x * 2
10000000 loops, best of 3: 50 ns per loop

In [14]: %timeit x = 1213; x = x << 1
10000000 loops, best of 3: 57.7 ns per loop

In [16]: %timeit x = 1213; x = x + x
10000000 loops, best of 3: 48.1 ns per loop
```
* 비트연산보다 덧셈이 더 빠르다.
* 여러가지 방식으로 시도해보자.

### 2. 파이썬의 내장 함수 및 표준 라이브러리를 최대한 활용한다.
* 직접 작성한 함수를 실행할 때의 오버헤드는 내장 함수보다 크다.

### 3. 반복문(for, while) 사용을 최대한 피한다.
* 같은 함수를 for문에서 여러 번 호출하는 것보다 데이터를 한꺼번에 함수에 넘기고 함수 안에서 반복 처리를 하는 쪽이 속도가 더 빠르다.
* Numpy는 이런 메커니즘을 ufunc에서 제공한다.
* 직접 작성한 함수를 for문에서 반복 호출하기 전에 ndarray와 ufunc의 조합으로 for문 사용을 피할 수 있는지 고려하자.

* for문과 리스트 컴프리헨션의 속도 차이 => 결론: 리스트 컴프리헨션을 쓰자.
```py
def list_by_for(x, y):
    z = []
    append = z.append
    for k in range(len(x)):
        append(x[k]*y[k] + 0.1*y[k])
    return z


def list_by_lc(x, y):
    return [a*b + 0.1*b for a, b in zip(x, y)]


if __name__ == '__main__':
    N1 = 1000000
    x = list(range(N1, 0, -1))
    y = list(range(N1))
    z1 = list_by_for(x, y)
    z2 = list_by_lc(x, y)

```

```
In [1]: %run -p -s cumulative lc_speed_test.py

ncalls  tottime  percall  cumtime  percall filename:lineno(function)
1    0.346    0.346    0.419    0.419 lc_speed_test.py:1(list_by_for)
1    0.000    0.000    0.175    0.175 lc_speed_test.py:9(list_by_lc)
```

* list_by_for는 0.419, list_by_lc는 0.175 가 걸렸다.

* for, while을 사용할 때 계산 속도에 있어 일반적으로 유의해야 할 점
    1. 속도상으로 확인된 병목에 대해서만 최적화할 것. 가장 안쪽 반복문만 최적화 할 것
    2. 내장 연산자나 내장 함수를 이용한 묵시적 반복이 for, while 보다 빠르다.
    3. for문 안에서 직접 작성한 파이썬 함수(lamba식 포함)를 호출하지 않는다. 인라인 전개를 직접하는 편이 훨씬 빠르다.
    4. 지역 변수는 전역 변수보다 빠르다. 전역상수를 반복문 안에서 사용해야 한다면 지역 변수에 복사하여 사용하는 것이 좋다.
    5. for문을 사용한 처리를 map(), filter(), functools.redece()로 바꿀 수 있다. 단, 내장 함수와 함께 사용할 수 없는 경우는 속도 개선이 보장되지 않는다.
    6. 반복문의 반복 횟수가 적은 경우에는 알고리즘 개선에 시간을 들이지 않아도 된다.
    7. 처리 속도를 정확히 측정한다. 파이썬 구현이나 버전에 따라 최적화되어 빠르게 동작하거나 예상보다 느릴 수 있다.
