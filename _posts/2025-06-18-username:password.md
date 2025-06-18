---
layout: single
title: "[Dreamhack] username:password@ write-up"
date: 2025-06-18 22:37 +0900
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

## [Dreamhack] username:password@

### 문제 설명

- url에 인증정보를 사용해 봤어요 🤪

<br>

### 풀이

접속하면 아래와 같이 로그인을 하라고 한다.

![image-20250617184153142](/images/2025-06-17-username:password/image-20250617184153142.png)

<br>

주요 소스 코드를 보면, flag 값은 /admin 경로에 들어가면 되는데 admin 계정으로 로그인을 해야 접속이 가능한 것으로 보인다. loginRequired에는 `HTTP Basic Authentication`을 사용하고 있다. /register 경로에는 의미있는 기능이 존재하지 않고, /report 경로에서 path 파라미터를 입력받아서 응답을 내려주고 있다.

```javascript
const genRanHex = size =>
  [...Array(size)].map(() => Math.floor(Math.random() * 16).toString(16)).join('');

const users = {
  'admin': genRanHex(16),
};


const loginRequired = basicAuth({
  authorizer: (username, password) => users[username] == password,
  unauthorizedResponse: 'Unauthorized',
});

const adminOnly = (req, res, next) => {
  if (req.auth?.user == 'admin') {
    return next();
  }
  return res.status(403).send('Only admin can access this resource');
};


app.get('/', (req, res) => {
  res.send('login with http://username:password@...');
});

app.get('/register', (req, res) => {
  res.send('Not implemented');
});

app.get('/report', loginRequired, (req, res) => {
  const path = req.query.path;

  if (!path) {
    return res.send("<form method='GET'>http://<input name='path' /><button>Submit</button></form>");
  }

  fetch(`https://admin:${users["admin"]}@${path}`)
    .then(() => res.send("Success"))
    .catch(() => res.send("Error"));
});

app.get('/admin', loginRequired, adminOnly, (req, res) => {
  res.send(flag);
});
```

<br>

HTTP Basic Authentication이란 username과 password를 `username:password` 로 만들어 base64 인코딩해준 후 아래와 같이 헤더를 포함하여 요청해야 한다. /report 경로에서 path 파라미터를 포함한 요청을 하기 위해서는 먼저 admin의 계정을 알아야 한다.

```
Authorization : Basic YWxpYwU6GFzc3dvcmQ==
```

<br>

admin의 비밀번호는 16자리의 랜덤한 16진수로 되어있는 것을 알 수 있다. `loginRequired`를 우회하기 위해서는 `users[username] == password` 에서 객체를 넣게 되면 자동으로 toString 메서드가 호출되면서 `[object Object]` 로 바뀐다. 하지만 username 부분에 아무 값이나 넣으면 안되고 모든 객체에서 사용할 수 있는 `__proto__` 를 username에 넣어주면 이를 우회할 수 있다.

```javascript
const genRanHex = size =>
  [...Array(size)].map(() => Math.floor(Math.random() * 16).toString(16)).join('');

const users = {
  'admin': genRanHex(16),
};

const loginRequired = basicAuth({
  authorizer: (username, password) => users[username] == password,
  unauthorizedResponse: 'Unauthorized',
});
```

<br>

아래와 같은 문자열을 base64로 인코딩하여 Authorization 헤더로 요청하면 401에러가 뜨지 않는 것을 볼 수 있다.

```
__proto__:[object Object]
```

![image-20250618213137257](/images/2025-06-17-username:password/image-20250618213137257.png)

<br>

이제 report 경로를 통해서 request bin을 이용해 해당 사이트로 요청을 날리게 되면 아래처럼 admin 계정의 인증 헤더가 base64로 인코딩되어 노출되어 있는 것을 알 수 있다.

![image-20250618221617852](/images/2025-06-17-username:password/image-20250618221617852.png)

<br>

복호화하게되면 아래처럼 admin 계정의 비밀번호가 나오게 되고 해당 토큰을 그대로 이용해 /admin 경로로 요청하면 flag 값을 얻을 수 있다.

![image-20250618221659721](/images/2025-06-17-username:password/image-20250618221659721.png)

![image-20250618221751914](/images/2025-06-17-username:password/image-20250618221751914.png)
