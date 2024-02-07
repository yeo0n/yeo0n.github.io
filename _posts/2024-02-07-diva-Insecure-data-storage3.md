---
layout: single
title: "[DIVA] Insecure Data Storage - Part 3"
date: 2024-02-07 20:50 +0900
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

## Insecure Data Storage

DIVA 앱의 Part3의 Insecure Data Storage의 액티비티로 들어가면 전과 같이 3rd party 서비스의 `name`과 `password`를 저장하는 액티비티인 것을 알 수 있다.

![image-20240207225502835](/images/2024-02-07-diva-Insecure-data-storage3/image-20240207225502835.png)

<br>

아래와 같이 `test3`과 `password3`의 계정을 생성했다.

![image-20240207230018109](/images/2024-02-07-diva-Insecure-data-storage3/image-20240207230018109.png)



<br>

jadx 도구로 AndroidManifest.xml 파일을 확인해보면 `jakhar.aseem.diva`의 경로에 이 액티비티의 소스 코드가 존재한다는 것을 알 수 있다.

![image-20240207225755893](/images/2024-02-07-diva-Insecure-data-storage3/image-20240207225755893.png)

<br>

### 소스 코드

액티비티의 소스 코드는 아래와 같고, 소스 코드를 분석해보면 `pin`을 SharedPreferences에 암호화하지 않고 저장하고 있다. 또한 `gotoNotes` 메서드는 `AccessControl3NotesActivity`라는 액티비티에 또 접근하는 것을 알 수 있다.

```java
    public void addPin(View view) {
        SharedPreferences spref = PreferenceManager.getDefaultSharedPreferences(this);
        SharedPreferences.Editor spedit = spref.edit();
        EditText pinTxt = (EditText) findViewById(R.id.aci3Pin);
        String pin = pinTxt.getText().toString();
        if (pin == null || pin.isEmpty()) {
            Toast.makeText(this, "Please Enter a valid pin!", 0).show();
            return;
        }
        Button vbutton = (Button) findViewById(R.id.aci3viewbutton);
        spedit.putString(getString(R.string.pkey), pin);
        spedit.commit();
        if (vbutton.getVisibility() != 0) {
            vbutton.setVisibility(0);
        }
        Toast.makeText(this, "PIN Created successfully. Private notes are now protected with PIN", 0).show();
    }

    public void goToNotes(View view) {
        Intent i = new Intent(this, AccessControl3NotesActivity.class);
        startActivity(i);
    }
}
```

<br>

`AccessControl3NotesActivity`의 소스코드는 아래와 같고, 전과 동일하게 `getText()` 함수와 `toString()` 함수를 통해서 암호화 없이 그대로 저장하는 걸 알 수 있다.

```java
/* loaded from: classes.dex */
public class AccessControl3NotesActivity extends AppCompatActivity {
    /* JADX INFO: Access modifiers changed from: protected */
    @Override // android.support.v7.app.AppCompatActivity, android.support.v4.app.FragmentActivity, android.support.v4.app.BaseFragmentActivityDonut, android.app.Activity
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_access_control3_notes);
    }

    public void accessNotes(View view) {
        EditText pinTxt = (EditText) findViewById(R.id.aci3notesPinText);
        Button abutton = (Button) findViewById(R.id.aci3naccessbutton);
        SharedPreferences spref = PreferenceManager.getDefaultSharedPreferences(this);
        String pin = spref.getString(getString(R.string.pkey), "");
        String userpin = pinTxt.getText().toString();
        if (userpin.equals(pin)) {
            ListView lview = (ListView) findViewById(R.id.aci3nlistView);
            Cursor cr = getContentResolver().query(NotesProvider.CONTENT_URI, new String[]{"_id", "title", "note"}, null, null, null);
            String[] columns = {"title", "note"};
            int[] fields = {R.id.title_entry, R.id.note_entry};
            SimpleCursorAdapter adapter = new SimpleCursorAdapter(this, R.layout.notes_entry, cr, columns, fields, 0);
            lview.setAdapter((ListAdapter) adapter);
            pinTxt.setVisibility(4);
            abutton.setVisibility(4);
            return;
        }
        Toast.makeText(this, "Please Enter a valid pin!", 0).show();
    }
}
```

<br>
확인해보기 위해 adb를 연결한 후, 로컬 저장소가 있는 `data/data/jakhar.aseem.diva` 경로에 들어가 시간대별로 변경된 파일 확인이 가능한 `ls -alR` 명령을 입력한다.

![image-20240207231056537](/images/2024-02-07-diva-Insecure-data-storage3/image-20240207231056537.png)

<br>
위와 같은 `uinfo306074722tmp` 파일이 생성되어 있는 것을 확인할 수 있고, 확인해보면 아래와 같이 계정이 평문으로 저장되어 있는 것을 확인할 수 있다.

![image-20240207232604654](/images/2024-02-07-diva-Insecure-data-storage3/image-20240207232604654.png)
