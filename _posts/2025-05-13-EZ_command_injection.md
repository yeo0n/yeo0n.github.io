---
layout: single
title: "[Dreamhack] EZ_command_injection write-up"
date: 2025-05-13 21:42 +0900
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

## [Dreamhack] EZ_command_injection

### 문제 설명

- ez command injection chall

<br>

### 풀이

문제 사이트에 접속해보면 Ping Result 부분으로 뭔가 결과를 출력해주는 것 같다.

![image-20250513214501695](/images/2025-05-13-EZ_command_injection/image-20250513214501695.png)

<br>

소스 코드를 살펴보면 /ping 경로에서 GET 메소드로 `host` 라는 파라미터를 받아와 `host`라는 변수에 저장한다. 그리고 host를 ipaddress.ip_address(host)를 통해 addr 변수에 다시 저장하고 있는데, 여기서 만약 host가 999.999.999.999와 같이 IP주소가 아니라면 예외가 발생한다. ip address인 것이 확인되면 ping 명령어의 인자로 들어가고, 예외가 발생하면 각각 어떤 오류인지 error_msg로 출력해주고 있다.

```python
#!/usr/bin/env python3
import subprocess
import ipaddress
from flask import Flask, request, render_template

app = Flask(__name__)

@app.route('/', methods=['GET'])
def index():
    return render_template('index.html')

@app.route('/ping', methods=['GET'])
def ping():
    host = request.args.get('host', '')
    try:
        addr = ipaddress.ip_address(host)
    except ValueError:
        error_msg = 'Invalid IP address'
        print(error_msg)
        return render_template('index.html', result=error_msg)

    cmd = f'ping -c 3 {addr}'
    try:
        output = subprocess.check_output(['/bin/sh', '-c', cmd], timeout=8)
        return render_template('index.html', result=output.decode('utf-8'))
    except subprocess.TimeoutExpired:
        error_msg = 'Timeout!!!!'
        print(error_msg)
        return render_template('index.html', result=error_msg)
    except subprocess.CalledProcessError:
        error_msg = 'An error occurred while executing the command'
        print(error_msg)
        return render_template('index.html', result=error_msg)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8000)

```

<br>

host 파라미터에 127.0.0.1과 127.0.0.1;ls 를 입력해보면 전자는 명령을 실행하는 동안 오류가 발생하였고, 후자는 IP address가 아니므로 Invalid IP address 문자열이 출력됐다.

![image-20250513215703297](/images/2025-05-13-EZ_command_injection/image-20250513215703297.png)

![image-20250513215748437](/images/2025-05-13-EZ_command_injection/image-20250513215748437.png)

<br>

먼저 `ipaddress.ip_address()` 함수를 우회해야 할 거 같다. `3221225985` 값을 넘겨주면 아래 이미지처럼 반환된다고 하여 host 파라미터에 넣어보면 아마도 싱글쿼터에 감싸져 있어서 IP가 아니라고 반환 되는 것 같다. 서칭을 좀 더 해보니 파이썬 3.9 미만 버전에서는 싱글쿼터에 감싸져 있어도 인식했지만 이상 버전부터는 안된다고 한다.

![image-20250513221310194](/images/2025-05-13-EZ_command_injection/image-20250513221310194.png)

![image-20250513221439646](/images/2025-05-13-EZ_command_injection/image-20250513221439646.png)

<br>

다음으로 ip_address에 대해서 IPv6를 살펴보면 %scope_id가 있는 경우 뒷부분에 %표기를 통해서 아래처럼 %1, %eth0 처럼 사용이 가능하다고 되어있다.

![image-20250513224348610](/images/2025-05-13-EZ_command_injection/image-20250513224348610.png)

<br>

아래와 같이 host 파라미터에 `::1%eth0`을 넣어보면 ip 검사가 통과되었고, 이를 통해 뒤에 ;ls를 붙여 command injection이 가능하다.

![image-20250513224454922](/images/2025-05-13-EZ_command_injection/image-20250513224454922.png)

![image-20250513224520826](/images/2025-05-13-EZ_command_injection/image-20250513224520826.png)

<br>

cat flag.txt 명령어를 넣어주면 flag 획득!

![image-20250513224727196](/images/2025-05-13-EZ_command_injection/image-20250513224727196.png)

