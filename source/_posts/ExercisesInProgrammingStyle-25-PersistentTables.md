---
title: 프로그래밍패턴 - 25.Persistent Tables
tags:
  - ExercisesInProgrammingStyle
  - 프로그래밍패턴
  - python
categories:
  - book
date: 2017-09-15 17:59:11
updated: 2017-09-15 17:59:11
layout:
---

* [프로그래밍패턴](http://wikibook.co.kr/programming-patterns/)을 읽고 공부하면서 쓰는 글입니다.

* code는 저자의 github(https://github.com/crista/exercises-in-programming-style) 에 있는 것을 typing만 했습니다. 

## Persistent Tables

* 간단하게 DB를 사용한 예제

* sqlite가 python과 원래 하나였던 것 처럼 사용하는 방법이 간결하고 편하다.

* `load_file_into_database`를 보면 함수 안에 함수를 만들고 바로 호출한다.

* 눈에 잘 들어오는 효과는 있는데 왜 이렇게 했을까?

* 상위 25개를 출력할 때 range(25)와 fetchone을 사용했다. 이것도 기억해두자.

* query에서 25개만 조회하여 fetchall을 사용하는건 어떨까?

```py
import sys, re, string, sqlite3, os


def create_db_schema(connection):
    c = connection.cursor()
    c.execute('''CREATE TABLE documents
                 (id INTEGER PRIMARY KEY AUTOINCREMENT, name)''')
    c.execute('''CREATE TABLE words (id, doc_id, value)''')
    c.execute('''CREATE TABLE characters (id, word_id, value)''')
    connection.commit()
    c.close()


def load_file_into_database(path_to_file, connection):
    def _extract_words(path_to_file):
        with open(path_to_file) as f:
            str_data = f.read()
        pattern = re.compile('[\W_]+')
        word_list = pattern.sub(' ', str_data).lower().split()

        with open('../stop_words.txt') as f:
            stop_words = f.read().split(',')
        stop_words.extend(list(string.ascii_lowercase))

        return [w for w in word_list if not w in stop_words]

    words = _extract_words(path_to_file)

    c = connection.cursor()
    c.execute("INSERT INTO documents (name) VALUES (?)", (path_to_file,))
    c.execute("SELECT id FROM documents WHERE name=?", (path_to_file,))
    doc_id = c.fetchone()[0]

    c.execute("SELECT MAX(id) FROM words")
    row = c.fetchone()
    word_id = row[0]
    if word_id == None:
        word_id = 0
    for w in words:
        c.execute("INSERT INTO words VALUES (?, ?, ?)", (word_id, doc_id, w))
        char_id = 0
        for char in w:
            c.execute("INSERT INTO characters VALUES (?, ?, ?)"
                , (char_id, word_id, char))
            char_id += 1
        word_id += 1
    connection.commit()
    c.close()


if not os.path.isfile('tf.db'):
    with sqlite3.connect('tf.db') as connection:
        create_db_schema(connection)
        load_file_into_database(sys.argv[1], connection)


with sqlite3.connect('tf.db') as connection:
    c = connection.cursor()
    c.execute('''SELECT value, COUNT(*) as C 
               FROM words GROUP BY value ORDER BY C DESC''')
    for i in range(25):
        row = c.fetchone()
        if row != None:
            print(f'{row[0]} - {row[1]}')

```