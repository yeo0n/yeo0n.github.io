---
layout: single
title: "[Dreamhack] simple-web-request write-up"
date: 2024-02-04 13:52 +0900
categories: 
    - WEB-write-up
tag: request
typora-root-url: ../
toc: true
toc_sticky: true
toc_label: 목차
author_profile: true
comment: true
sidebar:
    nav: "docs"
---



## 문제 설명

STEP 1~2를 거쳐 FLAG 페이지에 도달하면 플래그가 출력됩니다.  

모든 단계를 통과하여 플래그를 획득하세요. 플래그는 flag.txt 파일과 FLAG 변수에 있습니다.  

플래그 형식은 DH{...} 입니다.  

<br>

## 풀이

이 문제에서는 직접 웹 앱을 실행하고 Flask 웹 서버에 접속할 수 있도록 가이드를 알려주어 나도 한 번 해보려고 한다. 문제의 소스 코드는 다음과 같다.

![image-20240204135726014](/images/2024-02-04-simple-web-request/image-20240204135726014.png)

<br>

### 웹앱 실행 가이드

1. 터미널 창을 열고 `deploy` 디렉토리로 이동합니다.
2. Flask가 처음이라면 `pip install flask` 명령어로 Flask를 설치합니다.
3. `python app.py` 명령어로 파이썬 파일을 실행합니다.
4. 출력된 웹 주소를 복사하여 브라우저에서 접속합니다.

<br>

 VS code 편집기를 열어 이 `deploy` 폴더를 열어준 후 터미널 창에서 `python app.py` 명령어를 입력하면 그냥 바로 실행된다.

![image-20240204140252478](/images/2024-02-04-simple-web-request/image-20240204140252478.png)

<br>

터미널에 나온 주소를 브라우저 창에 입력하여도 되고 [Ctrl] 키를 누른 채로 URL 주소를 클릭하면 바로 접속할 수 있다. 하지만 해당 웹 서버에서 문제를 풀면 sample flag가 출력되니 문제는 해당 문제의 접속 URL로 들어가 풀어야 한다.

![image-20240204140325125](/images/2024-02-04-simple-web-request/image-20240204140325125.png)

<br>

### 소스 코드

```python
#!/usr/bin/python3
import os
from flask import Flask, request, render_template, redirect, url_for
import sys

app = Flask(__name__)

try: 
    # flag is here!
    FLAG = open("./flag.txt", "r").read()      
except:
    FLAG = "[**FLAG**]"


@app.route("/")
def index():
    return render_template("index.html")


@app.route("/step1", methods=["GET", "POST"])
def step1():

    #### 풀이와 관계없는 치팅 방지 코드
    global step1_num
    step1_num = int.from_bytes(os.urandom(16), sys.byteorder)
    ####

    if request.method == "GET":
        prm1 = request.args.get("param", "")
        prm2 = request.args.get("param2", "")
        step1_text = "param : " + prm1 + "\nparam2 : " + prm2 + "\n"
        if prm1 == "getget" and prm2 == "rerequest":
            return redirect(url_for("step2", prev_step_num = step1_num))
        return render_template("step1.html", text = step1_text)
    else: 
        return render_template("step1.html", text = "Not POST")


@app.route("/step2", methods=["GET", "POST"])
def step2():
    if request.method == "GET":

    #### 풀이와 관계없는 치팅 방지 코드
        if request.args.get("prev_step_num"):
            try:
                prev_step_num = request.args.get("prev_step_num")
                if prev_step_num == str(step1_num):
                    global step2_num
                    step2_num = int.from_bytes(os.urandom(16), sys.byteorder)
                    return render_template("step2.html", prev_step_num = step1_num, hidden_num = step2_num)
            except:
                return render_template("step2.html", text="Not yet")
        return render_template("step2.html", text="Not yet")
    ####

    else: 
        return render_template("step2.html", text="Not POST")

    
@app.route("/flag", methods=["GET", "POST"])
def flag():
    if request.method == "GET":
        return render_template("flag.html", flag_txt="Not yet")
    else:

        #### 풀이와 관계없는 치팅 방지 코드
        prev_step_num = request.form.get("check", "")
        try:
            if prev_step_num == str(step2_num):
        ####

                prm1 = request.form.get("param", "")
                prm2 = request.form.get("param2", "")
                if prm1 == "pooost" and prm2 == "requeeest":
                    return render_template("flag.html", flag_txt=FLAG)
                else:
                    return redirect(url_for("step2", prev_step_num = str(step1_num)))
            return render_template("flag.html", flag_txt="Not yet")
        except:
            return render_template("flag.html", flag_txt="Not yet")
            

app.run(host="0.0.0.0", port=8000)

```

<br>

### flag 조건

이번 문제는 되게 간단하다.  아래 소스 코드를 살펴보면 GET 메소드로 `param`과 `param2` 파라미터를 받고 이 인자 값들이 각각 `getget`과 `rerequest`이면 step2로 넘어갈 수 있다.

```python
@app.route("/step1", methods=["GET", "POST"])
def step1():

    #### 풀이와 관계없는 치팅 방지 코드
    global step1_num
    step1_num = int.from_bytes(os.urandom(16), sys.byteorder)
    ####

    if request.method == "GET":
        prm1 = request.args.get("param", "")
        prm2 = request.args.get("param2", "")
        step1_text = "param : " + prm1 + "\nparam2 : " + prm2 + "\n"
        if prm1 == "getget" and prm2 == "rerequest":
            return redirect(url_for("step2", prev_step_num = step1_num))
        return render_template("step1.html", text = step1_text)
    else: 
        return render_template("step1.html", text = "Not POST")
```

<br>

flag 엔드포인트에서 확인해보면 step2에서 인자가 각각 `pooost` , `requeeest`이면 flag가 나온다고 알려준다.

```python
@app.route("/flag", methods=["GET", "POST"])
def flag():
    if request.method == "GET":
        return render_template("flag.html", flag_txt="Not yet")
    else:

        #### 풀이와 관계없는 치팅 방지 코드
        prev_step_num = request.form.get("check", "")
        try:
            if prev_step_num == str(step2_num):
        ####

                prm1 = request.form.get("param", "")
                prm2 = request.form.get("param2", "")
                if prm1 == "pooost" and prm2 == "requeeest":
                    return render_template("flag.html", flag_txt=FLAG)
                else:
                    return redirect(url_for("step2", prev_step_num = str(step1_num)))
            return render_template("flag.html", flag_txt="Not yet")
        except:
            return render_template("flag.html", flag_txt="Not yet")
```

<br>

다음과 같이 Step1 엔드포인트에서 `getget`과 `rerequest` 값을 넣어주면 Step2 엔드포인트로 넘어가는 것을 확인할 수 있다.

![image-20240204142014698](/images/2024-02-04-simple-web-request/image-20240204142014698.png)

![image-20240204142031067](/images/2024-02-04-simple-web-request/image-20240204142031067.png)

<br>

Step2의 인자는 flag 엔드포인트 소스 코드에서 확인했듯이 `pooost`와 `requeeest` 를 넣어주면 flag를 획득할 수 있다.

![image-20240204142129767](/images/2024-02-04-simple-web-request/image-20240204142129767.png)

![image-20240204142149905](/images/2024-02-04-simple-web-request/image-20240204142149905.png)