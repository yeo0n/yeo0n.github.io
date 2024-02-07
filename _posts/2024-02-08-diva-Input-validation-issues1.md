---
layout: single
title: "[DIVA] Input Validation Issues - Part 1"
date: 2024-02-08 01:06 +0900
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

DIVA의 Input Validation Issues는 입력의 유효성을 검증하지 않아서 발생하는 취약점이다. 이에 대해서 간단히 확인해볼 수 있는 취약점은 흔히 알고 있는 XSS와 SQLi취약점이 존재할 확률이 높다. DIVA앱에서 7번째의 Input Validation Issues의 액티비티를 눌러 들어간다.

![image-20240208011326533](/images/2024-02-08-diva-Input-validation-issues1/image-20240208011326533.png)

<br>
접속하면 다음과 같이 user name을 설치하는 액티비티가 존재하는 것을 확인할 수 있다.

![image-20240208011342496](/images/2024-02-08-diva-Input-validation-issues1/image-20240208011342496.png)

<br>

이후 입력되는 값의 결과 로그를 확인하기 위해 adb를 사용해 `logcat` 명령어로 로그캣을 열어놓는다.

![image-20240208011446766](/images/2024-02-08-diva-Input-validation-issues1/image-20240208011446766.png)

<br>

DIVA앱에서 `'`싱글 쿼터를 입력하고 SEARCH 버튼을 4번정도 누르니 아래와 같이 에러 구문이 나오는 것을 확인할 수 있다.

![image-20240208011742356](/images/2024-02-08-diva-Input-validation-issues1/image-20240208011742356.png)

<br>
이제 싱글쿼터를 2개와 3개를 입력해보면  `token: ""`의 더블쿼터 사이와 `user = ''`의 싱글쿼터 사이에 입력되는 것을 볼 수 있는데 정상적인 SQL 쿼리는 아래의 `user` 뒤에 오는 싱글쿼터 사이에 문자열이 들어가는 것을 알 수 있다.

![image-20240208012104992](/images/2024-02-08-diva-Input-validation-issues1/image-20240208012104992.png)

<br>

따라서 싱글쿼터를 아래와 같이 우회하고 모든 계정을 검색해보면 SQLi 취약점이 있는 것을 알 수 있다.

![image-20240208012314702](/images/2024-02-08-diva-Input-validation-issues1/image-20240208012314702.png)

<br>

### 취약한 소스 코드

jadx 도구를 이용해 취약한 소스 코드를 살펴보면 소스 코드에서 직접 아래 계정들을 INSERT 문을 이용해 데이터를 생성하고, 가장 중요한 데이터가 삽입되는 SQL 쿼리는 `Cursor cr = this.mDB.rawQuery("SELECT * FROM sqliuser WHERE user = '" + srchtxt.getText().toString() + "'", null);` 인데 사용자가 입력되는 `srchtxt.getText().toString() 부분에서 아무런 검증을 하지 않아서 발생하는 것이다.

```java
package jakhar.aseem.diva;

import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.util.Log;
import android.view.View;
import android.widget.EditText;
import android.widget.Toast;

/* loaded from: classes.dex */
public class SQLInjectionActivity extends AppCompatActivity {
    private SQLiteDatabase mDB;

    /* JADX INFO: Access modifiers changed from: protected */
    @Override // android.support.v7.app.AppCompatActivity, android.support.v4.app.FragmentActivity, android.support.v4.app.BaseFragmentActivityDonut, android.app.Activity
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        try {
            this.mDB = openOrCreateDatabase("sqli", 0, null);
            this.mDB.execSQL("DROP TABLE IF EXISTS sqliuser;");
            this.mDB.execSQL("CREATE TABLE IF NOT EXISTS sqliuser(user VARCHAR, password VARCHAR, credit_card VARCHAR);");
            this.mDB.execSQL("INSERT INTO sqliuser VALUES ('admin', 'passwd123', '1234567812345678');");
            this.mDB.execSQL("INSERT INTO sqliuser VALUES ('diva', 'p@ssword', '1111222233334444');");
            this.mDB.execSQL("INSERT INTO sqliuser VALUES ('john', 'password123', '5555666677778888');");
        } catch (Exception e) {
            Log.d("Diva-sqli", "Error occurred while creating database for SQLI: " + e.getMessage());
        }
        setContentView(R.layout.activity_sqlinjection);
    }

    public void search(View view) {
        EditText srchtxt = (EditText) findViewById(R.id.ivi1search);
        try {
            Cursor cr = this.mDB.rawQuery("SELECT * FROM sqliuser WHERE user = '" + srchtxt.getText().toString() + "'", null);
            StringBuilder strb = new StringBuilder("");
            if (cr != null && cr.getCount() > 0) {
                cr.moveToFirst();
                do {
                    strb.append("User: (" + cr.getString(0) + ") pass: (" + cr.getString(1) + ") Credit card: (" + cr.getString(2) + ")\n");
                } while (cr.moveToNext());
            } else {
                strb.append("User: (" + srchtxt.getText().toString() + ") not found");
            }
            Toast.makeText(this, strb.toString(), 0).show();
        } catch (Exception e) {
            Log.d("Diva-sqli", "Error occurred while searching in database: " + e.getMessage());
        }
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

