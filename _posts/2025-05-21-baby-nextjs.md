---
layout: single
title: "[Dreamhack] baby-nextjs write-up"
date: 2025-05-21 01:12 +0900
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

## [Dreamhack] baby-nextjs

### 문제 설명

- use nextjs 👶

<br>

### 풀이

문제에 처음 접속하면 give me the flag라는 버튼이 있어, 눌러 보면 unavailable in production 문자열이 나온다.

![image-20250521011430496](/images/2025-05-21-baby-nextjs/image-20250521011430496.png)

![image-20250521011508594](/images/2025-05-21-baby-nextjs/image-20250521011508594.png)

<br>

page.js 페이지 소스 코드를 살펴보면 `use client`를 통해서 클라이언트에서 실행되는 컴포넌트라고 선언하고 있고, actions.js에서 `getFlag()` 함수를 가져온다. IS_PROD에서는 환경변수로 현재 실행 중인 환경이 `production` 인지 확인하고 true면 배포버전 false면 개발 모드이다. 개발 모드이면 `getFlag()` 함수를 통해서 flag를 출력해주는 코드이다.

```javascript
'use client';

import { useState } from 'react';
import { getFlag } from './actions';

const IS_PROD = process.env.NODE_ENV === 'production';

export default function Home() {
  const [message, setMessage] = useState('');

  return (
    <>
      {message && <p>{message}</p>}
      <button
        onClick={async () => {
          if (IS_PROD) {
            setMessage('unavailable in production');
            return;
          }
          const { message } = await getFlag();
          setMessage(message);
        }}
      >
        give me the flag
      </button>
    </>
  );
}

```

<br>

actions.js 파일의 소스 코드도 살펴보면, 서버 컴포넌트라는걸 선언하였고, import 문으로 js, path, url 모듈을 가져와서 사용하고 있다. `__dirname` 변수에는  `/app/action.js` 라면 `/app` 이 저장된다. 그리고 flag를 읽어오는 getFlag() 함수가 존재한다.

```javascript
'use server';

import { readFileSync } from 'node:fs';
import { resolve, dirname } from 'node:path';
import { fileURLToPath } from 'node:url';

const __dirname = dirname(fileURLToPath(import.meta.url));

export async function getFlag() {
  return { message: readFileSync(resolve(__dirname, '../../flag.txt'), 'utf8') };
}

```

<br>

layout.js 파일에서는 딱히 볼게 없다.

```javascript
export const metadata = {
  title: "Baby Next.js",
  description: "👶",
};

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body>
        {children}
      </body>
    </html>
  );
}

```

<br>

이번 문제에서 해당 버튼을 클릭하고 개발자 도구의 네트워크 및 burp에서 history를 분석해보면, js 파일을 불러오는 거 말고는 버튼을 눌렀을 때의 요청이 남아있지 않다. 따라서 처음엔 개발 모드 production을 어떻게 조작하여 우회할지 고민해보았지만, 검색하다보니 next.js의 13~14 버전에서 클라이언트 코드에서 백엔드 함수를 직접 호출하는 방식이 가능하다는 것을 알게 되었다.

<br>

next.js에서는 서버 함수마다 내부적으로 고유한 identifier를 생성하고 `_rsc` 또는 `_actions` 등의 비공개 API 엔드포인트로 호출한다. 또한 함수를 호출할 때 구조는 보통 아래와 같이 json을 사용하여 호출이 되고, Next-Action이라는 헤더에 Action_ID 값을 넣어 요청하게 된다. 따라서 번들에서 getFlag() 함수를 호출하는 Action ID가 존재함으로 이를 분석하여 찾아야 한다. 참고로 $$는 Next.js 내부에서 이 값이 Server Action ID임을 식별하기 위한 접두사이다.

```
POST /_rsc HTTP/1.1
Content-Type: application/json
Next-Action: Action_ID
...

["$$ACTION_ID", null]
```

<br>

burp 히스토리에서 요청된 js 소스 코드에서 Action_ID를 찾기 위해서 getFlag, flag, callServer 등 문자열로 찾아봐야 한다. 또한 이 Action_ID는 길이가 40인 유니크 문자열이며, callServer는 클라이언트에서 서버 함수를 직접 호출할 때 사용되는 함수로 이를 검색하여 찾을 수 있다.

![image-20250521210016708](/images/2025-05-21-baby-nextjs/image-20250521210016708.png)

<br>

이제 burp 요청에서 아래와 같이 _rsc 경로로 Action_ID를 가져와 요청하면 flag 값을 획득할 수 있다.

![image-20250521205721080](/images/2025-05-21-baby-nextjs/image-20250521205721080.png)