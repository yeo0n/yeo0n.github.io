---
layout: single
title: "[DIVA] Insecure Data Storage - Part 1"
date: 2024-02-05 21:10 +0900
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

## 로컬저장소 내 평문 저장된 중요 정보 취약점이란?

- 안드로이드 앱은 로컬 스토리지에 파일을 저장할 수 있다.
- 환경설정 정보, 외부 API 연동 및 인증을 위한 토큰 값, 연동 서버 정보 등이 로컬 스토리지에 저장된다.
- 루팅된 환경에서는 애플리케이션이 다른 애플리케이션의 디렉토리에 접근할 수 있으므로 데이터를 탈취할 수 있다.
- 중요 데이터가 평문 형태로 저장되어 제3자에게 유출될 경우 2차피해로 이어질 수 있다.

<br>

아래는 로컬 저장소에 어떤 저장소들이 있는지 나열한 것이다. 이것들 말고도 캐쉬 저장소나 등등  더 존재한다.

<br>

### 내부 저장소 (Internal Storage)

애플리케이션을 설치하게 되면 `/data/data/`경로에 설치한 애플리케이션과 동일한 디렉토리가 생성된다. 이 디렉토리는 생성한 애플리케이션에서만 사용 가능하고, 다른 앱에선 접근할 수 없다. 또한 애플리케이션이 제거되면 내부 저장소에 저장된 파일도 삭제된다. 일반적으로는 안전하지만 루팅된 에뮬레이터나 기기는 확인 가능하다.

![image-20240205212731670](/images/2024-02-05-diva-Insecure-data-storage1/image-20240205212731670.png)

<br>

### Shared Preferences

Shared  Preferences 객체는 `key-value` 가 포함된 파일을 가리키며, 앱의 환경설정이나 인증정보 등을 저장하는데 사용된다. 이 저장소는 `/data/data/[package.name]/shared_prefs/`경로에 존재하며 xml 확장자 파일로 저장된다. diva 디렉토리에는 Shared Prefs 폴더가 없어 InsecureBankv2 디렉토리에서 확인했다.

![image-20240205213719021](/images/2024-02-05-diva-Insecure-data-storage1/image-20240205213719021.png)

<br>

### SQLite Databases

모바일 환경에서 일반적으로 사용되는 경량 파일 기반 데이터베이스이다. 안드로이드 SDK는 SQLite 데이터베이스를 기본적으로 지원한다고 한다. 또한 특정 애플리케이션의 SQLite 데이터베이스에 저장된 데이터는 다른 애플리케이션에 접근할 수 없다. 경로는 `/data/data/[package_name]/databases/ ` 경로에 db 확장자 파일로 저장된다. 

![image-20240205215545996](/images/2024-02-05-diva-Insecure-data-storage1/image-20240205215545996.png)

<br>

### 외부 저장소 (External Storage)

외부 저장소는 안드로이드 기기에서 사용자가 직접 접근하고 읽고 쓸 수 있는 저장 공간이다. 주로 사진, 동영상, 문서 등의 파일을 저장하는 데 사용되며, 기기에 SD카드가 잇을 경우 SD 카드를 포함한다. 외부 저장소의 데이터는 앱 외부에서도 접근 가능하므로, 파일이나 미디어를 다른 앱과 공유할 수 있다. 따라서 민감한 정보는 여기에 저장하면 안된다. 애플리케이션을 제거해도 외부 저장소에 저장된 파일은 삭제되지 않는다. 경로는 `/sdcard` 또는 `/mnt/sdcard/` , `/storage/emulated/0`, `/storage/emulated/0/Android/data/[package_name]/files`경로에 저장된다.

![image-20240205220437522](/images/2024-02-05-diva-Insecure-data-storage1/image-20240205220437522.png)

<br>

DIVA 앱에서 3번 째에 있는 [INSECURE DATA STORAGE - PART 1]을 누르게 되면 3rd party service의 `name`과 `password`를 입력받고 저장하고 있다. 

![image-20240206002328511](/images/2024-02-05-diva-Insecure-data-storage1/image-20240206002328511.png)

<br>
여기서 이제 `/data/data/jakhar.assem.diva` 경로의 내부 저장소(Internal Storage) 설치 디렉토리로 이동해보자. 여기서 만약에 에뮬레이터나 단말기가 '루팅' 즉 루트 권한이 없을 경우 접근할 수 없다. 접근하면 아래와 같이 `cache, code_cache, databases, lib, shared_prefs` 폴더가 존재한다. 

![image-20240206013151448](/images/2024-02-05-diva-Insecure-data-storage1/image-20240206013151448.png)

<br>

`ls -alR` 명령을 입력하면 하위 디렉토리 경로에 있는 모든 파일과 폴더를 리스트 형태로 출력해준다. 이제 여기서 SAVE 버튼을 눌렀을 때의 시간을 찾아서 어떤 파일이 변경되었는지 찾을 수 있다. 살펴보면 아래의 `jakhar.assem.diva_preferences.xml` 파일이 변경된 것을 알 수 있다.

![image-20240206013928901](/images/2024-02-05-diva-Insecure-data-storage1/image-20240206013928901.png)

<br>

`jakhar.assem.diva_preferences.xml` 파일을 확인해보면 아래와 같이 3rd party 계정을 저장한 name과 password가 평문으로 그대로 노출되고 있는 것을 확인할 수 있다.

![image-20240206015338654](/images/2024-02-05-diva-Insecure-data-storage1/image-20240206015338654.png)

<br>

jadx 도구로 이 3번 째의 액티비티 소스 코드 확인해보면 아래와 같이 usr와 pwd 변수를 `getText()` 함수를 이용해 그대로 가져오고 `toString()`함수를 통해 문자열로 변환하여 그대로 노출해서 보여주는 것을 알 수 있다. 또한 `.getDefaultSharedPreferences` 함수를 사용하면서 spref에 commit하여 저 xml 파일이 저 위치에 있다는 것도 알 수 있다.

```java
/* loaded from: classes.dex */
public class InsecureDataStorage1Activity extends AppCompatActivity {
    /* JADX INFO: Access modifiers changed from: protected */
    @Override // android.support.v7.app.AppCompatActivity, android.support.v4.app.FragmentActivity, android.support.v4.app.BaseFragmentActivityDonut, android.app.Activity
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_insecure_data_storage1);
    }

    public void saveCredentials(View view) {
        SharedPreferences spref = PreferenceManager.getDefaultSharedPreferences(this);
        SharedPreferences.Editor spedit = spref.edit();
        EditText usr = (EditText) findViewById(R.id.ids1Usr);
        EditText pwd = (EditText) findViewById(R.id.ids1Pwd);
        spedit.putString("user", usr.getText().toString());
        spedit.putString("password", pwd.getText().toString());
        spedit.commit();
        Toast.makeText(this, "3rd party credentials saved successfully!", 0).show();
    }
}
```

<br>

### 대응 방안

- 로컬 저장소에 저장된 데이터는 노출될 우려가 있으므로 최소한의 정보만 저장한다.

- 중요 정보를 저장해야 하는 경우 안전한 암호화 알고리즘을 이용하여 저장한다. 

  





































