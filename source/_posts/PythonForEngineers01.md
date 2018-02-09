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

* 11장을 다 읽고 정리했다.
* 대부분 이해는 했다. 하지만 이해한 것과 체득해서 자유롭게 사용할 수 있는 것과는 다르다.
* 반복해서 읽어보고 연습해보자.

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

## 메모리의 효율적인 사용
* 파이썬은 가비지 컬렉션에의해 대부분의 경우 빠른 시간 내에 메모리가 해제되지만, 사용이 끝난 변수가 원시 객체(primitive object)일 때는 값을 저장하고 있는 메모리 영역을 캐시로 잠시 유지하는 경우가 있다.
* 메모리 크기가 문제가 될 수 있는 프로그램은 다음 사항을 고려한다.
    * 크기가 큰 데이터를 저장한 변수를 del로 삭제한다.
    * 메모리를 많이 사용하는 처리를 함수 안에서 수행하여 함수를 벗어난 뒤 자동적으로 해제되도록 한다.

### ndarray에서 메모리 절약하기
* ndarray의 불필요한 복제가 일어나지 않도록 하는 방법
* 참조를 만드는 것은 괜찮지만 최대한 사본을 만들지 않도록 한다.
    1. 원래 변수의 값이 바뀌는 연산자(in-place operator)를 사용한다.
    2. 되도록 배열의 변형(reshaping)이나 전치로 인한 묵시적 복사가 일어나지 않도록 한다.
    3. flatten과 ravel 중 ravel을 우선 사용한다.
    4. 사본을 생성할 때 가능하면 응용 인덱싱을 사용하지 않는다.

1. 원래 변수의 값이 바뀌는 연산자(in-place operator)를 사용한다.

```py
import time

import numpy as np


a = np.random.randn(100000)
print(f'original identity : {id(a)}')

ts = time.clock()
a *= 2
te = time.clock()
print(f'inplace: identity : {id(a)}, 실행 시간 : {te-ts:0.10}')

ts = time.clock()
a = a * 2
te = time.clock()
print(f'normal: identity : {id(a)}, 실행 시간 : {te-ts:0.10}')
```

```
original identity : 4485748256
inplace: identity : 4485748256, 실행 시간 : 7.4e-05
normal: identity : 4491277536, 실행 시간 : 0.000362
```
* in-place operator(`a *= 2`)로 연산하면 identity도 변경되지 않고 실행 시간도 짧다.
* 일반적으로 연산했을 경우(`a = a * 2`) `a * 2` 에 해당하는 사본이 만들어진 다음 사본이 a에 대입된다.

2. 되도록 배열의 변형(reshaping)이나 전치로 인한 묵시적 복사가 일어나지 않도록 한다.

```py
a = np.arange(100).reshape(10, 10)
b = a.reshape((1, -1))
c = a.T.reshape((1, -1))
```
* ndarray를 변형하거나 전치했을 때 메모리상의 데이터 순서가 바뀌지 않는다면 복사가 일어나지 않는다.
* 이 경우에 메모리도 절약할 수 있고 실행시간도 단축할 수 있다.
* a와 b는 동일한 데이터다.

```
In [3]: a = np.arange(100).reshape(10, 10)

In [4]: %timeit a.reshape((1, -1))
420 ns ± 12.7 ns per loop (mean ± std. dev. of 7 runs, 1000000 loops each)
In [6]: %timeit a.T.reshape((1, -1))
1.02 µs ± 10.9 ns per loop (mean ± std. dev. of 7 runs, 1000000 loops each)
```
* a.reshape((1, -1))는 420ns, a.T.reshape((1, -1))은 1.02us
* a.reshape가 휠씬 빠르다.
* 참고로 ns(nanosecond)는 10^-9, us(mircosecond)는 10^-6

3. flatten과 ravel 중 ravel을 우선 사용한다.
```py
In [2]: a = np.arange(100).reshape(10, 10)

In [3]: %timeit a.flatten()
The slowest run took 113.45 times longer than the fastest. This could mean that an intermediate result is being cached.
1000000 loops, best of 3: 700 ns per loop

In [4]: %timeit a.ravel()
The slowest run took 25.49 times longer than the fastest. This could mean that an intermediate result is being cached.
1000000 loops, best of 3: 201 ns per loop
```
* flatten, ravel은 모두 행렬을 1차원으로 만드는 기능을 한다.
* flatten은 항상 사본을 만든다.
* ravel은 불필요한 사본을 만들지 않는다. (사본을 만들 수도 있다는 것 같은데 언제 만들어지는지 모르겠다.)
* ravel은 사본을 만들지 않기 때문에 새로운 변수에 할당한 값을 변경하여도 원본도 변경된다.
* 속도는 ravel이 더 빠르다.


4. 사본을 생성할 때 가능하면 응용 인덱싱을 사용하지 않는다.
```py
In [12]: n, d = 1000, 100

In [13]: a = np.random.random_sample((n, d))

In [14]: %timeit a[::10]
The slowest run took 28.34 times longer than the fastest. This could mean that an intermediate result is being cached.
1000000 loops, best of 3: 202 ns per loop

In [15]: %timeit a[np.arange(0, n, 10)]
The slowest run took 10.24 times longer than the fastest. This could mean that an intermediate result is being cached.
100000 loops, best of 3: 10.6 µs per loop
```
* 응용 인덱싱이 정확이 어떤 의미인지 잘 모르겠다.
* a[::10]은 a의 일부를 참조하는 뷰
* a[np.arnage(0, n, 10)]은 a의 일부를 복사한 사본
* 참조하는 뷰쪽이 매우 빠르다.

### 참고
* numpy 내장함수
    * rand: N차원 배열의 난수 발생
    * randn: 표준 정규분포에 따른 N차원 난수 발생
    * randint(low[, hign, size]): low 이상 high 미만의 정수형 난수 발생
    * rand_integer(low[, hign, size]): low 이상 high 이하의 정수형 난수 발생
    * random_sample([size]): 0.0 이상 1.0 미만의 실수형 난수 발생
        * random([size]), ranf([size]), sample([size]) 모두 같은 함수
    * choice(a[, size, replace, p]): 주어진 1차원 배열 a에서 샘플 추출
    * bytes(length): 바이트형 난수 발생

## 프로파일러 활용
* IPython을 활용한 프로파일링은 별도로 공부할 예정.
1. IPython을 사용하지 않고 함수 프로파일링 하기
2. 프로파일링 결과를 시각화
3. IPython을 사용하지 않고 라인 프로파일링 하기

### 1. IPython을 사용하지 않고 함수 프로파일링 하기
```py
import cProfile
import numpy as np
import pstats


def is_prime(a):
    a = abs(a)
    if a == 2:
        return True
    if a < 2 or a & 1 == 0:
        return False
    return pow(2, a-1, a) == 1


def mysum(N):
    return np.arange(1, N+1).sum()


def task1(N):
    out = []
    append = out.append
    for k in range(1, N+1):
        if is_prime(k):
            append(k)
    
    a = mysum(N)
    return [out, a]


def task2(N):
    return np.sqrt(np.arange(1, N+1))


def main():
    task1(10000)
    task2(10000)


if __name__ == '__main__':
    cProfile.run('main()', filename='main.prof')
    sts = pstats.Stats('main.prof')
    sts.strip_dirs().sort_stats('cumulative').print_stats()
```

```
Fri Feb  9 10:10:06 2018    main.prof
         26262 function calls in 0.013 seconds
   Ordered by: cumulative time
   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.000    0.000    0.013    0.013 {built-in method builtins.exec}
        1    0.000    0.000    0.013    0.013 <string>:1(<module>)
        1    0.000    0.000    0.013    0.013 spd_Profile.py:34(main)
        1    0.002    0.002    0.013    0.013 spd_Profile.py:19(task1)
    10000    0.004    0.000    0.010    0.000 spd_Profile.py:6(is_prime)
     4999    0.006    0.000    0.006    0.000 {built-in method builtins.pow}
        1    0.000    0.000    0.001    0.001 spd_Profile.py:15(mysum)
        2    0.001    0.000    0.001    0.000 {built-in method numpy.core.multiarray.arange}
    10000    0.000    0.000    0.000    0.000 {built-in method builtins.abs}
        1    0.000    0.000    0.000    0.000 spd_Profile.py:30(task2)
        1    0.000    0.000    0.000    0.000 {method 'sum' of 'numpy.ndarray' objects}
        1    0.000    0.000    0.000    0.000 _methods.py:31(_sum)
        1    0.000    0.000    0.000    0.000 {method 'reduce' of 'numpy.ufunc' objects}
     1251    0.000    0.000    0.000    0.000 {method 'append' of 'list' objects}
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
```
* cProfile.run() 메소드를 이용하여 main()을 실행시키고 분석했다.
* 분석 결과는 main.prof 파일에 저장된다.
* 프로파일 결과가 저장된 파일을 Stats 클래스를 이용하여 정렬하고 다시 출력한다.
* 위 결과는 누적시간(cumulative)을 기준으로 정렬했다.
* task1과 task2의 누적시간을 확인할 수 있다.

* 각 항목의 의미는 다음과 같다.
    * ncalls: 호출 횟수
    * tottime: 해당 함수에서 소비된 합계 시간(서브 함수에서 소비된 시간은 제외)
    * percall: tottime/ncalls
    * cumtime: 해당 함수와 모든 서브 함수에서 소비된 누적시간
    * percall: cumtime을 호출 횟수로 나눈 값
    * filename:lineno(function): 해당 함수의 파일명/줄번호/함수명

### 2. 프로파일링 결과를 시각화
* http://jiffyclub.github.io/snakeviz/
* 설치: pip install snakeviz
* command line interface: snakeviz main.prof

### 3. IPython을 사용하지 않고 라인 프로파일링 하기
```py
if __name__ == '__main__':
    from line_profiler import LineProfiler
    prf = LineProfiler()
    # prf.add_function(is_prime)
    prf.runcall(is_prime, 999)
    prf.print_stats()
```

```
Timer unit: 3.01854e-07 s

Total time: 6.94264e-06 s
File: spd_Profile.py
Function: is_prime at line 6

Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
     6                                           def is_prime(a):
     7         1          8.0      8.0     34.8      a = abs(a)
     8         1          2.0      2.0      8.7      if a == 2:
     9                                                   return True
    10         1          2.0      2.0      8.7      if a < 2 or a & 1 == 0:
    11                                                   return False
    12         1         11.0     11.0     47.8      return pow(2, a-1, a) == 1
```
* IPython을 사용하여 라인 프로파일링을 할 수도 있다.(%lprun 사용)
* is_prime() 함수가 가장 많은 시간이 걸린다는 것을 알고 있다.
* runcall() 메소드를 이용하여 is_price 함수에 인자를 전달한다.
* 각 항목의 의미는 다음과 같다.
    * Line: 해당 파일의 라인 수
    * Hits: 해당 라인이 실행된 횟수
    * Time: Timer unit을 단위로 해당 라인을 실행하는데 걸린 시간
    * Per Hit: Timer unit을 단위로 해당 라인을 실행하는데 걸린 평균 시간
    * % Time: 해당 함수가 실행된 시간을 %로 표시
    * Line Contents: 실제 소스 코드

## 병렬처리
* C로 구현된 파이썬의 기준 구현체로 취급받는 CPython에는 GIL(Golbal Interperter Lock)이라는 매커니즘이 있다.
* GIL은 메모리 관리 등에 대한 저수준 처리 구현을 간단하게 하기 위한 것이다.
* 스레드 자체는 여러 개가 함께 존재할 수 있지만 스레드 안에서 일어나는 처리는 병렬처리할 수 없게 되어 있다.
* 하지만 Numpy로 병렬 처리할 할 수 있고 SIMD를 사용하면 싱글 스레드 안에서 병렬 처리를 할 수도 있다.

### SIMD(Single Instruction Multiple Data)
* 연산자 하나를 여러 데이터에 병렬로 적용하는 방법
* 대부분의 CPU는 SIMD 연산을 할 수 있다.
* Numpy나 SciPy가 SIMD를 지원한다.

### 스레드와 멀티스레드 적용
* 애플리케이션이 실행되면 프로세스 1개가 생성되고 이 프로세스가 여러 개의 스레드를 만든 뒤 이들 스레드에 처리를 나눠준다.
* 같은 프로세스에 속하는 스레드는 메모리상에 로드된 데이터나 힙, 라이브러리와 같은 정보를 공유할 수 있다.
* 파이썬에서는 스레드를 명시적으로 생성하지 않는 한 스레드는 1개밖에 생성되지 않는다.

* 외부 기억 장치로 부터 데이터를 읽어 온다거나, 네트워크 소켓에 대한 읽고 쓰기 등 비교적 대기 시간이 긴 I/O처리에 멀티스레드를 적용하면 대기시간 감소로 전체 실행 시간이 단축된다.

### 멀티프로세스 사용하기
* 프로세스란 프로그램의 실행 단위를 말한다.
* 윈도우에서는 확장자 .exe를 실행하면 이 파일 하나에 1개의 프로세스가 생성된다.
* 프로세스는 작업을 실행하는 데 필요한 모든 자원을 확보단다.
* 프로세스간 메모리 공유는 MPI(Message Passing Interface)등의 수단을 사용한다.
* 멀티프로세스를 활용하면 가장 먼저 만들어진 프로세스에서 다른 프로세스를 띄우고, 여기서 실행된 계산 결과를 원래 프로세스로 돌려준다.
* 프로세스를 만드는 오버헤드를 고려하면 처리 내용의 규모가 일정 이상되지 않으면 효과가 낮거나 역효과가 난다.
* multiprocessing은 프로세스 여러 개를 만들 수 있는 파이썬 표준 라이브러리다.
* concurrent.futures는 비동기 실행 고수준 인터페이스를 제공하는 파이썬 표준 라이브러리다. 멀티스레드 및 멀티프로세스 처리를 거의 비슷한 API로 제공한다.

```py
import concurrent.futures
import math
import time

PRIMES = [
    112272535095293,
    112582705942171,
    112272535095293,
    115280095190773,
    115797848077099,
    1099726899285419
]


def is_prime(n):
    if n % 2 == 0:
        return False
    sqrt_n = int(math.floor(math.sqrt(n)))
    for i in range(3, sqrt_n+1, 2):
        if n % i == 0:
            return False
    return True


def main1():
    ts = time.clock()
    with concurrent.futures.ProcessPoolExecutor() as executor:
        ans = list(executor.map(is_prime, PRIMES))
    print(f'multiprocess: {time.clock()-ts:{10}.{5}}')
    print(ans)


def main2():
    ts = time.clock()
    ans = list(map(is_prime, PRIMES))
    print(f'single process: {time.clock()-ts:{10}.{5}}')
    print(ans)


if __name__ == '__main__':
    main1()
    main2()
```

```
multiprocess:     1.1678
[True, True, True, True, True, False]
single process:     3.2465
[True, True, True, True, True, False]
```