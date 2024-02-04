---
layout: single
title: "[wargame.kr] out of money write-up"
date: 2024-02-04 17:16 +0900
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

## 문제 설명

드림이가 선물을 준비하려하는데 돈이 없네요...  

그럼 무에서 유를 창조해볼까요?  

Note : 음수의 값이 허용된다면 그 행동의 반대를 하게 됩니다.  

<br>

## 풀이

<br>

문제 페이지에 접속하면 이름을 적고 로그인 하는 폼이 존재한다. 

![image-20240204172142338](/images/2024-02-04-out-of-money/image-20240204172142338.png)
### 소스 코드

- app.py

```python
from flask import Flask, session, redirect, url_for, request, render_template
from threading import Thread
from util import get_price, deposit, liquidate

app = Flask(__name__)
app.secret_key = "[REDACTED]"

@app.route("/", methods=['GET', 'POST'])
def main():
    if request.method == 'POST' and request.form['name'] != "":
        session['name'] = request.form['name']

        session['DHH'] = 0.0
        session['DHC'] = 0.0
        session['DHD'] = 0.0

        session['debt_DHH'] = 0.0

        session['col_DHC'] = 0.0
        session['depo_DHC'] = 0.0
        session['depo_DHD'] = 0.0
        session['debt_DHD'] = 0.0

        return redirect(url_for('main'))

    if 'name' in session:
        return render_template("lobby.html", session=session)
    else:
        return render_template("login.html")

@app.route("/santa", methods=['GET', 'POST'])
def santa():
    return render_template("santa.html", session=session, message="")

@app.route("/santa/lend", methods=['POST'])
def santa_lend():
    value = float(request.form['value'])

    if session['debt_DHH'] + value >= 10000.0:
        return render_template("santa.html", session=session, message="그만 빌려욧!")
    if session['DHH'] + value < 0.0:
        return render_template("santa.html", session=session, message="더 갚으시게요...?")

    session['DHH'] += value
    session['debt_DHH'] += value
    return render_template("santa.html", session=session, message="대출완료!")

@app.route("/santa/flag", methods=['GET'])
def santa_flag():
    if session['DHH'] >= 1000.0:
        if session['debt_DHH'] == 0.0:
            return render_template("flag.html")
        else:
            return render_template("santa.html", session=session, message="빚을 먼저 값으세욧!")
    return render_template("santa.html", session=session, message="드핵코인이 없어욧!")

@app.route("/santa/change", methods=['POST'])
def santa_change():
    frm = int(request.form['from'])
    to = int(request.form['to'])
    value = float(request.form['value'])

    if value < 0:
        return render_template("santa.html", session=session, message="어디서 음수만큼 바꾸려고!")

    tbl = ['DHH', 'DHC', 'DHD']
    if frm in [0, 1, 2] and to in [0, 1, 2]:
        frm = tbl[frm]
        to = tbl[to]

        if session[frm] < value:
            return render_template("santa.html", session=session, message="가지고 있는 코인이 그만큼 없어욧!")

        if frm != to:
            frm_price = get_price(frm)
            to_price = get_price(to)

            to_balance = value * frm_price / to_price

            session[frm] -= value
            session[to] += to_balance
        return render_template("santa.html", session=session, message="교환 완료!")
    return render_template("santa.html", session=session, message="다른건 교환 못합니다!")

@app.route("/dream", methods=['GET'])
def dream():
    return render_template("dream.html", session=session, message="")

@app.route("/dream/collateral", methods=['POST'])
def dream_col():
    value = float(request.form['value'])

    if value < 0:
        if session['debt_DHD'] == 0.0:
            session['DHC'] += session['col_DHC']
            session['col_DHC'] = 0.0
            return render_template("dream.html", session=session, message="담보 반환 완료!")
        else:
            return render_template("dream.html", session=session, message="빚을 먼저 값으세욧!")
    if session['DHC'] - value < 0.0:
        return render_template("dream.html", session=session, message="가지고 있는 드냥코인이 부족합니다!")

    session['DHC'] -= value
    session['col_DHC'] += value
    return render_template("dream.html", session=session, message="담보 확인!")

@app.route("/dream/deposit", methods=['POST'])
def dream_deposit():
    type = int(request.form['type'])
    value = float(request.form['value'])

    if type not in [0, 1]:
        return render_template("dream.html", session=session, message="드핵코인, 드냥코인만 예금할 수 있습니다!")

    tbl = ['DHC', 'DHD']
    type = tbl[type]
    if session[type] - value < 0:
        return render_template("dream.html", session=session, message="가지고 있는 코인이 부족합니다!")
    if session["depo_" + type] + value < 0:
        return render_template("dream.html", session=session, message="에금한 코인보다 더 뺄수는 없습니다!")
    session[type] -= value
    session["depo_" + type] += value
    deposit(type, value)
    return render_template("dream.html", session=session, message="예금완료")

@app.route("/dream/lend", methods=['POST'])
def dream_loan():
    value = float(request.form['value'])

    dhc_price = get_price('DHC')
    dhd_price = get_price('DHD')

    max_lend = session['col_DHC'] * dhc_price / dhd_price * 0.8

    print(max_lend)

    if session['DHD'] + value < 0.0:
        return render_template("dream.html", session=session, message="더 갚으시게요...?")
    if max_lend < value:
        return render_template("dream.html", session=session, message="그만큼 빌리기에는 담보가 부족합니다!")

    session['DHD'] += value
    session['debt_DHD'] += value

    return render_template("dream.html", session=session, message="대출 완료!")

@app.route("/logout")
def logout():
    session.pop('name', None)
    return redirect(url_for('main'))

import time
def loop_liquid():
    while True:
        time.sleep(2)
        liquidate()

if __name__ == '__main__':
    t1 = Thread(target = loop_liquid, daemon=True)
    t1.start()
    app.run(host="0.0.0.0")

```

<br>

- util.py

```python
dhc_balance = 1000.0
dhd_balance = 1000.0

def get_price(name):
    if name == "DHH":
        return 1.0
    if name == "DHC":
        return 1.0
    if name == "DHD":
        return dhc_balance * get_price("DHC") / dhd_balance

def deposit(name, value):
    global dhc_balance, dhd_balance
    if name == "DHC":
        dhc_balance += value
    if name == "DHD":
        dhd_balance += value

def liquidate():
    pass
    # 차익거래 후, dhc_balance * dhc_price == dhd_balance * dhd_price
    # 만들어짐
    # 나만 이득 못봐!

```

<br>
<br>

먼저 santa/flag 엔드포인트를 살펴보면 이 문제의 용어로 드핵코인이 1000.0 보다 크거나 같고, 빌린 드핵코인이 0.0이면 flag가 출력되는 것을 알 수 있다.

```python
@app.route("/santa/flag", methods=['GET'])
def santa_flag():
    if session['DHH'] >= 1000.0:
        if session['debt_DHH'] == 0.0:
            return render_template("flag.html")
        else:
            return render_template("santa.html", session=session, message="빚을 먼저 값으세욧!")
    return render_template("santa.html", session=session, message="드핵코인이 없어욧!")

```

<br>

dream이란 임의의 이름으로 로그인하면 다음과 같이 산타 사설 거래소와 드림 유동성 풀이 존재한다.

![image-20240204173334765](/images/2024-02-04-out-of-money/image-20240204173334765.png)

<br>

다음과 같이 1000 DHH를 빌리면 드핵 코인과 빌린 드핵코인이 1000씩 올라갔다. 따라서 아까 Flag의 조건인 드핵코인이 1000이고 빌린 드핵코인을 0으로 만든 후 Flag 구매 버튼을 누르면 될듯하다.

![image-20240204173420885](/images/2024-02-04-out-of-money/image-20240204173420885.png)

<br>

먼저 1000 DHH를 대출받았다. 빌린 드핵코인도 1000.0 DHH로 되어 있을 것이다.

![image-20240204175149192](/images/2024-02-04-out-of-money/image-20240204175149192.png)

<br>

담보를 내기 위해선 DHH를 DHC로 바꿔줘야 한다. 이제 드림 유동성 풀에서 1000을 모두 담보로 낸다. 아래에선 문제를 먼저 풀고 작성하여 저렇게 되어있다.. 

![image-20240204175418678](/images/2024-02-04-out-of-money/image-20240204175418678.png)

<br>

이제 DHD를 빌리면 되는데 500씩 쪼개서 받으니 드멍코인이 500씩 채워진다. 이를 4번 해서 2000으로 땡긴 후에 빌린 드핵코인이 1000으로 되어 있을 텐데, 드멍코인 2000을 드핵코인으로 바꾼 후 무담보 대출 입력 폼에서 -1000을 하여 갚게 되면 드핵코인이 1000이 되고 빌린 드핵 코인이 0이 되어 flag를 획득할 수 있다.

![image-20240204175934956](/images/2024-02-04-out-of-money/image-20240204175934956.png)