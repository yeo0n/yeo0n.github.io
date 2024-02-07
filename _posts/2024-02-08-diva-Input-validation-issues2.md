---
layout: single
title: "[DIVA] Input Validation Issues - Part 2"
date: 2024-02-08 01:37 +0900
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

## 입력 유효성 검사 문제

DIVA앱의 8번째에 있는 Input Validation Issues 취약점의 Part2의 액티비티로 접속하면 URL을 입력하는 폼이 존재한다.

![image-20240208013912491](/images/2024-02-08-diva-Input-validation-issues2/image-20240208013912491.png)

<br>입력폼에 `https://www.google.com` 의 구글 주소를 입력해보면 아래에 구글 홈페이지를 보여준다.

![image-20240208014403565](/images/2024-02-08-diva-Input-validation-issues2/image-20240208014403565.png)

<br>
이제 파일도 읽는지 테스트하기 위해서 sdcard의 외부 저장소에서 아래와 같이 `Hello World`를 test.txt 파일에 저장했다.

![image-20240208014644477](/images/2024-02-08-diva-Input-validation-issues2/image-20240208014644477.png)

<br>
다시 DIVA 앱에서 `file:///` 의 파일스키마를 사용해서 `sdcard/test.txt` 파일을 VIEW하면 Hello World!가 출력된 것을 볼 수 있다.

![image-20240208014743674](/images/2024-02-08-diva-Input-validation-issues2/image-20240208014743674.png)

<br>
따라서 전에 있던 로컬저장소 내 평문 저장 취약점이 있었던 `/data/data/jakhar.aseem.diva/shared_prefs` 경로의 `jakhar.aseem.diva_preferences.xml`파일을 읽어오기 위해서 파일 스키마로 읽을 수 있는지 확인할 수 있다.

```text
file:///data/data/jakhar.aseem.diva/shared_prefs/jakhar.aseem.diva_preferences.xml
```

<br>

확인해보면 아래와 같이 로컬저장소 내의 내부저장소 파일을 읽어올 수 있는 걸 알 수 있다.

![image-20240208015324028](/images/2024-02-08-diva-Input-validation-issues2/image-20240208015324028.png)

<br>

### 취약한 소스 코드

소스 코드는 아래와 같이 `wview.loadUrl(uriText.getText().toString());`에서 따로 사용자 입력의 검증을 수행하지 않아 시스템의 내부 파일까지 읽을 수 있다. 즉 이 응용프로그램이 시스템의 모든 파일을 읽을 수 있는 권한을 가지게 되는 것이다.

```java
/* loaded from: classes.dex */
public class InputValidation2URISchemeActivity extends AppCompatActivity {
    /* JADX INFO: Access modifiers changed from: protected */
    @Override // android.support.v7.app.AppCompatActivity, android.support.v4.app.FragmentActivity, android.support.v4.app.BaseFragmentActivityDonut, android.app.Activity
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_input_validation2_urischeme);
        WebView wview = (WebView) findViewById(R.id.ivi2wview);
        WebSettings wset = wview.getSettings();
        wset.setJavaScriptEnabled(true);
    }

    public void get(View view) {
        EditText uriText = (EditText) findViewById(R.id.ivi2uri);
        WebView wview = (WebView) findViewById(R.id.ivi2wview);
        wview.loadUrl(uriText.getText().toString());
    }
}
```

<br>

### 대응 방안

1. 입력 검증
   - 엄격한 검증 기법을 사용하여 사용자 입력을 검증해야 한다. 입력 길이 등을 제한해 악의적인 데이터는 거부될 수 있도록 한다.
2. 출력 정보에 대한 보안
   - XSS 공격을 방지하기 위해 출력 정보에 대한 보안을 필수이다.
3. 맥락별 검증
   - 데이터 맥락에 기반한 검증을 수행해 경로 순회 등의 공격으로부터 예방한다.
4. 안전한 코딩 관행
   - 매개변수화된 쿼리 및 명령문을 사용하는 등 보안 코딩 방법에 따라야 한다.
5. 정기적인 보안 테스트
   - 침투 테스트 및 코드 검토 등 정기적인 보안 평가를 수행한다. 