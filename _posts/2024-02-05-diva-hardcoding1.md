---
layout: single
title: "[DIVA] Hardcoding Issues - Part 1"
date: 2024-02-05 20:30 +0900
categories: 
    - Android
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

## 하드코딩된 중요 정보 노출 취약점이란?

- 하드코딩이란 소스 코드 내에 데이터가 직접 하드코딩 되어있는 경우를 말한다. 
- 안드로이드 앱은 디컴파일을 통한 소스 코드 확인이 가능하므로 하드코딩된 중요 정보를 꼭 삭제하여야 한다. 

<br>

### 디컴파일 도구

예전에 했던 방법으로는 classes.dex를 dex2jar를 이용해 jar 파일로 변환한 후 jd-gui나 bytecodeviewer로 실습을 하였었지만, jadx라는 도구가 나타나면서 이런 복잡한 과정을 거치지 않아도 된다.  

jadx 도구는 apk 파일만 추출하거나 저장되어 있는 경우 자동으로 디컴파일하므로 분석하는 시간을 크게 단축할 수 있다. 

<br>

DIVA 앱에서 Hardcoding Issues - Part 1을 클릭한다. 

<img src="/images/2024-02-05-diva-hardcoding1/image-20240205203856490.png" alt="image-20240205203856490"  />

<br>

아무 문자열인 1234를 입력하고 엔터나 ACCESS 버튼을 누르면 아래와 같이 접근이 거부되었다는 문자열이 출력된다.

![image-20240205204003625](/images/2024-02-05-diva-hardcoding1/image-20240205204003625.png)


<br>
하드코딩된 중요 정보 노출 취약점은 소스 코드 내에 하드코딩 되어있기 때문에 jadx 디컴파일 도구를 이용해 액티비티를 먼저 찾아준다. 이 액티비티는 아래와 같은 경로에 존재한다.

![image-20240205204210995](/images/2024-02-05-diva-hardcoding1/image-20240205204210995.png)

<br>

액티비티의 소스 코드는 아래와 같다. 

```java
package jakhar.aseem.diva;

import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.widget.EditText;
import android.widget.Toast;

/* loaded from: classes.dex */
public class HardcodeActivity extends AppCompatActivity {
    /* JADX INFO: Access modifiers changed from: protected */
    @Override // android.support.v7.app.AppCompatActivity, android.support.v4.app.FragmentActivity, android.support.v4.app.BaseFragmentActivityDonut, android.app.Activity
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_hardcode);
    }

    public void access(View view) {
        EditText hckey = (EditText) findViewById(R.id.hcKey);
        if (hckey.getText().toString().equals("vendorsecretkey")) {
            Toast.makeText(this, "Access granted! See you on the other side :)", 0).show();
        } else {
            Toast.makeText(this, "Access denied! See you in hell :D", 0).show();
        }
    }
}
```

<br>

소스 코드를 분석하면 아래 equals 함수를 통해 `vendorsecretkey`의 문자열과 같은지 비교하고 있다. 아래 코드를 통해서 이 액티비티의 키 값에 `vendorsecretkey`를 입력하면 아래의 `Access granted!` 문자열이 나타날 것이다.

```java
public void access(View view) {
        EditText hckey = (EditText) findViewById(R.id.hcKey);
        if (hckey.getText().toString().equals("vendorsecretkey")) {
            Toast.makeText(this, "Access granted! See you on the other side :)", 0).show();
        } else {
            Toast.makeText(this, "Access denied! See you in hell :D", 0).show();
        }
    }
```

<br>

위처럼 하드코딩된 중요 정보 노출 취약점을 통해서 실습을 진행할 수 있었다. 

![image-20240205204819820](/images/2024-02-05-diva-hardcoding1/image-20240205204819820.png)

<br>

### 하드코딩된 중요 정보 확인

- 소스 코드에서 텍스트 검색을 통해 다음과 같은 키워드 검색

  ```text
  id, pw, username, password, passwd, key, secret, admin, root, decrypt, encrypt, aes 등등 ...
  ```

<br>

### 대응 방안

- 중요 정보가 소스 코드 내에 문구, 주석, 변수 값 등의 형식으로 하드코딩되어 있지 않도록 제거한다.
- 개발기에서 사용한 계정, 암호화 키값 등이 배포할 때에는 제거되도록 주의를 기울여 개발 및 배포한다.
- 중요정보가 소스코드 내에서 사용되어야 한다면, 암호화 기법을 사용해 보호한다.



























