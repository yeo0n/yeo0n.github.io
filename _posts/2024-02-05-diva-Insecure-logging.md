---
layout: single
title: "[DIVA] INSECURE LOGGING 불충분한 로깅 문제"
date: 2024-02-05 20:10 +0900
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

## INSECURE LOGGING(불충분한 로깅 문제)

불충분한 로깅 문제의 취약점은 앱이 실행되는 과정에서 logcat과 주요 패킷 정보, 기타 로그 정보에서 중요 정보가 평문으로 전송되기 때문에 다른 사용자에게 노출되어 위협이 될 수 있는지 점검하여야 한다.

<br>

Nox 에뮬레이터를 사용해 DIVA 앱을 열어준 후 cmd를 열어 `nox_adb shell` 명령을 통해 Nox 에뮬레이터와 adb를 연결할 수 있다. `nox_adb shell`을 사용하려면 Nox 에뮬레이터에서 루트 옵션을 활성화해준 후 빌드 번호를 연타하여 USB 디버깅 모드 활성화를 하고, 환경변수 설정해주면 사용할 수 있다.

![image-20240205194743418](/images/2024-02-05-diva-Insecure-logging/image-20240205194743418.png)

![image-20240205194701345](/images/2024-02-05-diva-Insecure-logging/image-20240205194701345.png)

<br>

1번을 선택해 불충분한 로깅 문제의 액티비티로 접속하면 입력하는 창이 나온다. 

![image-20240205195206317](/images/2024-02-05-diva-Insecure-logging/image-20240205195206317.png)

<br>
`logcat`이라는 adb 명령어를 이용해 로그를 확인할 수 있다. adb를 연결한 cmd 창에서 logcat을 입력해주자. DIVA 앱에서 123456을 입력하고 CHECK OUT 버튼을 누르면 아래와 같이 logcat에서 `credit card number`가 평문으로 노출된 것을 확인할  수 있다. 

![image-20240205195508001](/images/2024-02-05-diva-Insecure-logging/image-20240205195508001.png)

<br>

단말이 만약 악성코드에 의해 감염된다면 logcat 정보를 악의적인 사용자에게 노출될 가능성이 높다. 취약한 코드를 확인해보기 위해서 jadx 디컴파일 도구를 이용해서 확인해볼 수 있다. jadx 디컴파일 도구는 프로그램을 열고 apk 파일을 드래그만 해주면 되기 때문에 편하다는 장점이 있다. `AndroidManifest.xml` 파일을 살펴보면 다음과 같이 `LogActivity`의 액티비티의 경로를 볼 수 있다.

![image-20240205200308123](/images/2024-02-05-diva-Insecure-logging/image-20240205200308123.png)

<br>LogActivity의 소스 코드는 다음과 같다.

```java
package jakhar.aseem.diva;

import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.util.Log;
import android.view.View;
import android.widget.EditText;
import android.widget.Toast;

/* loaded from: classes.dex */
public class LogActivity extends AppCompatActivity {
    /* JADX INFO: Access modifiers changed from: protected */
    @Override // android.support.v7.app.AppCompatActivity, android.support.v4.app.FragmentActivity, android.support.v4.app.BaseFragmentActivityDonut, android.app.Activity
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_log);
    }

    public void checkout(View view) {
        EditText cctxt = (EditText) findViewById(R.id.ccText);
        try {
            processCC(cctxt.getText().toString());
        } catch (RuntimeException e) {
            Log.e("diva-log", "Error while processing transaction with credit card: " + cctxt.getText().toString());
            Toast.makeText(this, "An error occured. Please try again later", 0).show();
        }
    }

    private void processCC(String ccstr) {
        RuntimeException e = new RuntimeException();
        throw e;
    }
}
```

<br>

로그캣에 노출되어 지는 소스 코드를 살펴보면 아래와 같은데, `cctxt.getText().toString()` 부분이 암호화가 전혀 되지 않는다는 걸 알 수 있다.

```java
Log.e("diva-log", "Error while processing transaction with credit card: " + cctxt.getText().toString());
```

<br>

### 불충분한 로깅 취약점의 다른 예시

- 로그인, 인증 실패, 권한 설정 등 주요 기능 수행에 대한 로깅이 없는 경우
- 일정 주기로 로그에 대한 백업 절차가 없는 경우
- 로깅 및 모니터링이 필요한 부분을 명확하게 구분해서 로깅하지 않아, 불명확한 로깅 및 모니터링을 하는 경우

<br>

### 대응 방안

- 모든 로그인, 접근 제어, 인증 실패에 대해 로깅을 하고 정기적인 백업을 통해 보관
- 로그 관리 솔루션 등을 활용하기 위해 적절한 형식으로 로깅이 생성되는지 확인
- 의심스러운 활동을 감지하고 신속하게 대응할 수 있도록 임계치를 설정하고 모니터링
- 침해 사고 대응 및 복구 계획 수립

























