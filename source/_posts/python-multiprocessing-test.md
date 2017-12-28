---
title: python multiprocessing 테스트
tags:
  - multiprocessing
categories:
  - python
date: 2017-11-08 16:09:30
updated: 2017-11-08 16:09:30
---

# multiprocessing test

* DB 데이터를 로드하여 파일로 만드는 프로그램
* 한번에 여러개의 파일을 만드는데 병렬처리를 적용하면 어떨까 궁금해서 적용
* multiprocess, concurrent 등의 모듈이 있던데 일단 multiprocess 부터
* 참고
    * https://beomi.github.io/2017/07/05/HowToMakeWebCrawler-with-Multiprocess/

## multiprocessing 미적용코드
```py
import time

import db_connection
import period_generator

def main():
    start_time = time.time()
    db_con = db_connection.connect_db()
    cursor = db_con.cursor()
    table_name = 'DB_TABLE'
    date_from = '20160101'
    date_to = '20171231'
    for start, end in period_generator.get_from_to_month(date_from, date_to):
    #     print(start, end)
        file_name = f'{table_name}_{start}_{end}.DAT'
        query = f"""
        select
        BASE_DATE,BOND_SECTOR_CODE,BOND_MAT_CODE,
        rtrim(to_char(TRADE_CNT, 'FM99999999999999999990.9999999999'), '.'),
        rtrim(to_char(TRADE_QTY, 'FM99999999999999999990.9999999999'), '.'),
        rtrim(to_char(TRADE_AMT, 'FM99999999999999999990.9999999999'), '.'),
        INPUT_DTIME
        from {table_name}
        where BASE_DATE > '{start}'
        and BASE_DATE <= '{end}'
        """
        cursor.prepare(query)
        cursor.execute(None)
        rows = cursor.fetchall()
        with open(file_name, 'wt') as f:
            for row in rows:
                data = [str(item) for item in row]
                data = list(map(lambda item: item.replace('None', ''), data))
            #print(data)
                f.write('|'.join(data))
                f.write('\n')
    print(f'--- {time.time() - start_time} sec ---')


if __name__ == '__main__':
    main()
```

```py
# period_generator.py 공통모듈

import calendar
from datetime import datetime, timedelta
from dateutil.relativedelta import relativedelta
def get_from_to_month(date_from, date_to):
    next_month = start_date = datetime.strptime(date_from, '%Y%m%d').date()
    end_date = datetime.strptime(date_to, '%Y%m%d').date()
    while True:
        if (next_month >= end_date):
            return
        start_date = next_month
        next_month = next_month + relativedelta(months=1)
        yield (datetime.strftime(start_date, '%Y%m%d'),
               datetime.strftime(next_month, '%Y%m%d'))

if __name__ == '__main__':
    for start, end in get_from_to_month('20160101', '20171201'):
        print(start, end)
```

## multiprocessing 적용
```py

from multiprocessing import Pool
import time

import db_connection
import period_generator


def make_data_to_file(period):
    table_name = 'DB_TABLE'
    start, end = period

    db_con = db_connection.connect_db()
    cursor = db_con.cursor()
    file_name = f'{table_name}_{start}_{end}.DAT'
    query = f"""
    select
    BASE_DATE,BOND_SECTOR_CODE,BOND_MAT_CODE,
    rtrim(to_char(TRADE_CNT, 'FM99999999999999999990.9999999999'), '.'),
    rtrim(to_char(TRADE_QTY, 'FM99999999999999999990.9999999999'), '.'),
    rtrim(to_char(TRADE_AMT, 'FM99999999999999999990.9999999999'), '.'),
    INPUT_DTIME
    from {table_name}
    where BASE_DATE > '{start}'
    and BASE_DATE <= '{end}'
    """
    cursor.prepare(query)
    cursor.execute(None)
    rows = cursor.fetchall()
    with open(file_name, 'wt') as f:
        for row in rows:
            data = [str(item) for item in row]
            data = list(map(lambda item: item.replace('None', ''), data))
        #print(data)
            f.write('|'.join(data))
            f.write('\n')


def main():
    start_time = time.time()
    date_from = '20160101'
    date_to = '20171231'
    pool = Pool(processes=8)
    pool.map(make_data_to_file,
             period_generator.get_from_to_month(date_from, date_to))
    print(f'--- {time.time() - start_time} sec ---')


if __name__ == '__main__':
    main()

```

## 실행결과
* 미적용
```
$ python db_data_to_csv.py
--- 74.71347045898438 sec ---

$ python db_data_to_csv.py
--- 62.96029543876648 sec ---
```

* multiprocess 적용
```
# multi(process==4)
$ python db_data_to_csv_multi.py
--- 22.79927968978882 sec ---

# multi(process==8)
$ python db_data_to_csv_multi.py
--- 14.889488935470581 sec ---

$ python db_data_to_csv_multi.py
--- 13.742374181747437 sec ---
```

* 약 4.7배 빨라졌다.

## 결론
* 단순한 query로 여러 파일을 다수 생성할 경우 꼭 multiprocessing을 사용하자.

## 과제
* query를 매개변수를 이용하여 변경할 수 있도록 처리
* multiprocessing 공부
* concurrent 공부
