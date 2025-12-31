---
layout: single
title: "[mobilehacking.kr] ProjectApp writeup"
date: 2025-12-31 00:20 +0900
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

## [mobilehacking.kr] ProjectApp writeup

<br>

### 😁 문제 설명

- First Android Application Project

<br>

### ✏️ 풀이

mobilehacking.kr의 처음 문제인 ProjectApp을 선택해보자.

<img src="/images/2025-12-31-android-ProjectApp/image-20251230233725317.png" alt="image-20251230233725317" style="zoom:33%;" />

<br>

아래 [prob Download] 버튼을 통해서 `ProjectApp.apk` 파일을 다운받을 수 있다.

<img src="/images/2025-12-31-android-ProjectApp/image-20251230233752336.png" alt="image-20251230233752336" style="zoom:33%;" />

<br>

아래 adb install 명령어를 이용해 `ProjectApp.apk` 파일을 스마트폰에 설치하였다.![image-20251230234507202](/images/2025-12-31-android-ProjectApp/image-20251230234507202.png)

<br>

스마트폰에서 확인해보면 ProjectApp이 잘 설치되었다. 아래 화면은 실제 루팅된 디바이스 기기에 scrcpy를 이용해 화면에 미러링된 화면이다.

![image-20251230234605378](/images/2025-12-31-android-ProjectApp/image-20251230234605378.png)

<br>

앱을 실행해보면 Enter Serial 문자열로 시리얼 번호를 확인하는 로직인 걸 알 수 있다.

![image-20251230234623850](/images/2025-12-31-android-ProjectApp/image-20251230234623850.png)

<br>

jadx-gui를 이용해 먼저 AndroidManifest.xml 파일부터 확인해보면 패키지명은 `com.ctf.projectapp` 이라는 것을 알 수 있다.

![image-20251230235150317](/images/2025-12-31-android-ProjectApp/image-20251230235150317.png)

<br>

MainActivity를 먼저 확인하기 위해 jadx-gui에서 소스코드 -> com -> ctf.projectapp -> MainActivity로 들어가 소스 코드를 확인할 수 있다.

![image-20251230235952523](/images/2025-12-31-android-ProjectApp/image-20251230235952523.png)

<br>

소스 코드를 분석해보면 `serialEditText` 변수를 선언하였고, 앱에서 가장 처음 실행되는 메소드인 `onCreate` 를 보면 `serialEditText` 변수에 `findViewById` 를 통해 폼 안에 사용자가 입력한 문자열을 가져오고 있다. 또한 `setonClickListener` 를 통해서 check 버튼을 눌렀을 때에 `checkSerial()` 메소드가 실행되도록 되어있다.

```java
public class MainActivity extends AppCompatActivity {
    private EditText serialEditText;

    protected void onCreate(Bundle bundle) {
        super.onCreate(bundle);
        setContentView(C0536R.layout.activity_main);
        this.serialEditText = (EditText) findViewById(C0536R.id.serialEditText);
        ((Button) findViewById(C0536R.id.checkButton)).setOnClickListener(new View.OnClickListener() { // from class: com.ctf.projectapp.MainActivity.1
            @Override // android.view.View.OnClickListener
            public void onClick(View view) {
                MainActivity.this.checkSerial();
            }
        });
    }
```

<br>

`checkSerial()` 메소드에서는 `decodeSecret()` 메소드 결과와 같은지 확인하고 맞다면 Correct! 문자열이 뜬다고 되어있다. 이를 위해선 `decodeSecret()` 메소드를 먼저 분석해야 한다.

```java
    public void checkSerial() {
        if (decodeSecret().equals(this.serialEditText.getText().toString())) {
            showAlert("Correct!");
        } else {
            showAlert("Incorrect!");
        }
    }
```

<br>

`decodeSecret()` 메소드를 분석하면 가장 처음 `inputStreamOpenRawResource` 변수에 `getResources().openRawResource(C0536R.raw.secret);` 결과 값을 담고 있다. 이 값을 알기 위해서는  `openRawResource()` 메소드의 파일 처리 방식을 알아야 한다. Java에서는 `/res` 경로 아래에 `raw` 폴더를 생성하고 사용할 파일을 저장할 수 있는데, 이때 `openRawResource()` 메소드를 이용하여 읽을 수 있다. 다만 읽기 전용이다. `return new String(Base64.decode(new String(bArr), 0));` 에서 base64 인코딩된 문자열을 디코딩하므로 디코딩도 해주어야 한다!

```java
    private String decodeSecret() throws Resources.NotFoundException, IOException {
        try {
            InputStream inputStreamOpenRawResource = getResources().openRawResource(C0536R.raw.secret);
            byte[] bArr = new byte[inputStreamOpenRawResource.available()];
            inputStreamOpenRawResource.read(bArr);
            inputStreamOpenRawResource.close();
            return new String(Base64.decode(new String(bArr), 0));
        } catch (IOException e) {
            e.printStackTrace();
            return "";
        }
    }
```

<br>

jadx-gui에서 리소스 -> res -> raw 경로에 가보면 secret.txt 파일이 있고 base64로 인코딩된 문자열을 확인할 수 있다.

![image-20251231001741052](/images/2025-12-31-android-ProjectApp/image-20251231001741052.png)

<br>

해당 문자열을 디코딩하고 [SERIAL CHECK] 버튼을 누르면 Correct! 문자열을 확인할 수 있고, 해당 flag를 문제 설명 창에서 넣어주면 점수를 획득할 수 있다.

![image-20251230235806261](/images/2025-12-31-android-ProjectApp/image-20251230235806261.png)

![image-20251230235826430](/images/2025-12-31-android-ProjectApp/image-20251230235826430.png)

![image-20251230235914144](/images/2025-12-31-android-ProjectApp/image-20251230235914144.png)