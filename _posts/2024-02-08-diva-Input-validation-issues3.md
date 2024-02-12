---
layout: single
title: "[DIVA] Input Validation Issues - Part 3"
date: 2024-02-08 18:19 +0900
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

DIVA의 마지막 챌린지인 Input Validation Issues - Part3이다. 입력 유효성을 검증하지 않으면 웹 해킹에서도 사용되는 XSS, CSRF, 파일 업로드, SQL injection 등과 같은 취약점들과도 연계될 수 있다. 

![image-20240208182113505](/images/2024-02-08-diva-Input-validation-issues3/image-20240208182113505.png)

<br>이 액티비티를 접속하면 Launch Code를 입력하는 곳이 존재한다. 

![image-20240208182542701](/images/2024-02-08-diva-Input-validation-issues3/image-20240208182542701.png)

<br>
이 액티비티의 소스 코드를 먼저 살펴보면 다음과 같다. private를 통해서 Divajni의 클래스를 `djni` 이름으로 불러오고 있다. 다음으로 `initiateLaunchSequence`함수를 사용해 비교하고 있다.

```java
/* loaded from: classes.dex */
public class InputValidation3Activity extends AppCompatActivity {
    private DivaJni djni;

    /* JADX INFO: Access modifiers changed from: protected */
    @Override // android.support.v7.app.AppCompatActivity, android.support.v4.app.FragmentActivity, android.support.v4.app.BaseFragmentActivityDonut, android.app.Activity
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_input_validation3);
        this.djni = new DivaJni();
    }

    public void push(View view) {
        EditText cTxt = (EditText) findViewById(R.id.ivi3CodeText);
        if (this.djni.initiateLaunchSequence(cTxt.getText().toString()) != 0) {
            Toast.makeText(this, "Launching in T - 10 ...", 0).show();
        } else {
            Toast.makeText(this, "Access denied!", 0).show();
        }
    }
}
```

<br>

DivaJni의 클래스의 소스 코드는 아래와 같고 JNI(Java Native Interface)를 사용하여 네이티브 코드로 `access`와 `initiateLaunchSequence`가 정의되어 있는 것을 알 수 있다.

```java
package jakhar.aseem.diva;

/* loaded from: classes.dex */
public class DivaJni {
    private static final String soName = "divajni";

    public native int access(String str);

    public native int initiateLaunchSequence(String str);

    static {
        System.loadLibrary(soName);
    }
}
```

<br>
Ghidra를 이용해서 네이티브 소스 코드를 분석할 수 있다. 전에 다른 문제들을 풀었다면 apktool을 이용해 diva 앱을 디컴파일 하였을 것이다. 디컴파일하면 아래와 같은 폴더가 생성되는데 lib -> armeabi 폴더에 so 파일이 있다.

![image-20240208185801849](/images/2024-02-08-diva-Input-validation-issues3/image-20240208185801849.png)

![image-20240208185903304](/images/2024-02-08-diva-Input-validation-issues3/image-20240208185903304.png)



<br>
이제 Ghidra를 실행시킨 후에 프로젝트를 생성하고 so 파일을 임포트한 후, GUI 안에서 so 파일을 더블클릭한다.

![image-20240208190049766](/images/2024-02-08-diva-Input-validation-issues3/image-20240208190049766.png)

<br>

OK-> OK 그대로 누르고 아래와 같이 Symbol Tree의 Filter에서 위에서 분석했던 initalte... 함수를 입력해주고, 나오는 소스 파일을 클릭한다.

![image-20240208190125323](/images/2024-02-08-diva-Input-validation-issues3/image-20240208190125323.png)

<br>

그럼 오른쪽에 디컴파일된 소스 코드가 바로 보일 것이다. 소스 코드를 분석해보면 strcpy 함수와 strncmp를 이용해 비교를 하고 있고, `.dotdot` 문자열을 입력하면 `Launching in T - 10 ...` 문자열을 확인할 수 있다. 

![image-20240208190242245](/images/2024-02-08-diva-Input-validation-issues3/image-20240208190242245.png)

![image-20240208191509759](/images/2024-02-08-diva-Input-validation-issues3/image-20240208191509759.png)



<br>중요한 부분은 strcpy 함수를 통해 `local_lc`에 28바이트가 설정되어 있는데, 이를 버퍼오버플로우 시키기 위해서는 뒤에 널 배열인 4바이트가 존재한다. 따라서 32바이트의 문자 길이를 입력하면 버퍼오버플로우가 발생하게 되서 응용 프로그램이 종료되는 것을 볼 수 있다.  

영어 한 문자는 1바이트 이므로 A 32개를 입력하면 오버플로우가 발생한다.

![image-20240208195859496](/images/2024-02-08-diva-Input-validation-issues3/image-20240208195859496.png)



![image-20240208195906683](/images/2024-02-08-diva-Input-validation-issues3/image-20240208195906683.png)

