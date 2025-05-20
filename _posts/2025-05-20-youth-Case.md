---
layout: single
title: "[Dreamhack] youth-Case write-up"
date: 2025-05-20 01:03 +0900
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

## [Dreamhack] youth-Case

### 문제 설명

- Bypass 👨‍💻filter

<br>

### 풀이

문제 사이트에 접속하면 hi guest 문자열이 출력되고 있다.

![image-20250520010600726](/images/2025-05-20-youth-Case/image-20250520010600726.png)

<br>

소스 코드를 살펴보면 nginx와 nodeJS로 구성되어 있다.

![image-20250520010717830](/images/2025-05-20-youth-Case/image-20250520010717830.png)

<br>

아래는 nginx.conf 파일의 소스 코드이고, /shop과 /shop/ 경로에 대해서 접근을 막고 있고, 나머지 경로에 대해서는 app:3000/으로 app 폴더 하위에 있는 nodeJS 서버로 리버 프록시를 설정하고 있다.

```
events {
    worker_connections  1024;
}

http {
    server {
        listen 80;
        listen [::]:80;
        server_name  _;
        
        location = /shop {
            deny all;
        }

        location = /shop/ {
            deny all;
        }

        location / {
            proxy_pass http://app:3000/;
        }

    }

}
```

<br>

shop 경로로 접근해보면 403 에러가 나오는 것을 볼 수 있다. app.js 소스 코드를 확인해보자.

![image-20250520132648695](/images/2025-05-20-youth-Case/image-20250520132648695.png)

<br>

app.js 소스 코드를 분석해보면 ag.js 파일을 불러 와서 words 변수에 저장하고 있고, search 함수를 살펴보면 words 인자에 배열을 받아와서 name 속성을 뽑아와 leg 인자의 대문자와 ===로 엄격한 비교를 해서 리턴하게 된다. `/` 경로에서는 hi guest 가 출력되고, `/shop` 경로에서는 POST 메소드로 leg 값이 flag면 403을 반환하고, leg값이 flag가 아니면 search 함수를 통해 json 응답으로 반환해준다.

```javascript
const words = require("./ag")
const express = require("express")

const PORT = 3000
const app = express()
app.set('case sensitive routing', true)
app.use(express.urlencoded({ extended: true }))

function search(words, leg) {
    return words.find(word => word.name === leg.toUpperCase())
}

app.get("/",(req, res)=>{
    return res.send("hi guest")
})

app.post("/shop",(req, res)=>{
    const leg = req.body.leg.toLowerCase()

    if (leg == 'flag'){
        return res.status(403).send("Access Denied")
    }

    const obj = search(words,leg)

    if (obj){
        return res.send(JSON.stringify(obj))
    }

    return res.status(404).send("Nothing")
})

app.listen(PORT,()=>{
    console.log(`[+] Started on ${PORT}`)
})
```

<br>

먼저 nginx.conf 파일에서 /shop 경로를 아예 차단하고 있기 때문에 app.js에서 /shop 경로에 대한 코드를 구현한다고 해도 접근이 불가하기 때문에 이를 우회해야 할 것 같다.

<br>

nginx.conf 파일을 보면 아래 마지막에 `/` 문자가 붙어있으므로 nginx가 URI를 정규화하여 서버로 보내기 때문에 우회가 거의 불가능하다. 또한 app.js 파일을 확인해보면 대소문자를 통한 우회도 방어하고 있다.

```
#nginx.conf
proxy_pass http://app:3000/;

app.js
app.set('case sensitive routing', true)
```

<br>

여기서 Response의 Server 응답 헤더를 살펴보면 nginx 1.22.0 버전으로 해당 버전에서  WAF Proxy를 우회할 수 있는 방법을 찾아야 한다. 

![image-20250520231900078](/images/2025-05-20-youth-Case/image-20250520231900078.png)<br>

아래 페이지에서 살펴보면 NodeJS의 express에서 nginx 버전이 1.22.0 버전일 때 `\xA0` 문자를 통해서 우회가 가능하다고 되어있다.

https://book.hacktricks.wiki/en/pentesting-web/proxy-waf-protections-bypass.html

<br>

이 문자가 우회 가능하다고 되는데 해결하기 위해서 burp 설정에서 [burp user] Settings > Ui > Message editor > character sets > Display as raw bytes 설정을 켜주고 \x가 16진수 이기 때문에 a0을 넣어서 아래처럼 요청을 보내보면 name이 DRAG인 값이 잘 요청되어 반환된 것을 알 수 있다.

![image-20250520234140010](/images/2025-05-20-youth-Case/image-20250520234140010.png)

<br>

이제 아래 부분에서 `leg=='flag'` 를 우회하여야 한다. 비교하는 구문에서 `toLowerCase()` 함수에서는 예를 들어 튀르키에어  İ와 같은 문자가 toLowerCase 함수를 거치면 i가 되어 다른 나라들의 문자 유니코드를 사용하여 우회가 가능하다.  아래 문자 중 우리가 구하고 싶은 문자는 FLAG 이기 때문에 ` ﬂ (%EF%AC%82)` 이 인코딩을 넣어주도록 하자.

```python
app.post("/shop",(req, res)=>{
    const leg = req.body.leg.toLowerCase()

    if (leg == 'flag'){
        return res.status(403).send("Access Denied")
    }

    const obj = search(words,leg)

    if (obj){
        return res.send(JSON.stringify(obj))
    }

    return res.status(404).send("Nothing")
})
```

```
[223] ß (%C3%9F).toUpperCase() => SS (%53%53)
[304] İ (%C4%B0).toLowerCase() => i̇ (%69%307)
[305] ı (%C4%B1).toUpperCase() => I (%49)
[329] ŉ (%C5%89).toUpperCase() => ʼN (%2bc%4e)
[383] ſ (%C5%BF).toUpperCase() => S (%53)
[496] ǰ (%C7%B0).toUpperCase() => J̌ (%4a%30c)
[7830] ẖ (%E1%BA%96).toUpperCase() => H̱ (%48%331)
[7831] ẗ (%E1%BA%97).toUpperCase() => T̈ (%54%308)
[7832] ẘ (%E1%BA%98).toUpperCase() => W̊ (%57%30a)
[7833] ẙ (%E1%BA%99).toUpperCase() => Y̊ (%59%30a)
[7834] ẚ (%E1%BA%9A).toUpperCase() => Aʾ (%41%2be)
[8490] K (%E2%84%AA).toLowerCase() => k (%6b)
[64256] ﬀ (%EF%AC%80).toUpperCase() => FF (%46%46)
[64257] ﬁ (%EF%AC%81).toUpperCase() => FI (%46%49)
[64258] ﬂ (%EF%AC%82).toUpperCase() => FL (%46%4c)
[64259] ﬃ (%EF%AC%83).toUpperCase() => FFI (%46%46%49)
[64260] ﬄ (%EF%AC%84).toUpperCase() => FFL (%46%46%4c)
[64261] ﬅ (%EF%AC%85).toUpperCase() => ST (%53%54)
[64262] ﬆ (%EF%AC%86).toUpperCase() => ST (%53%54)
```

<br>

![image-20250521000526334](/images/2025-05-20-youth-Case/image-20250521000526334.png)

