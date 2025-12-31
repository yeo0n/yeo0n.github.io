---
layout: single
title: "[mobilehacking.kr] Mobile Baby Analysis3 writeup"
date: 2026-01-01 00:34 +0900
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

## [mobilehacking.kr] Mobile Baby Analysis3 writeup

### 문제설명

- Mobile App Reversing

<br>

아래 prob Download 버튼을 통해 apk 파일을 다운받도록 하자.

![image-20251231105932087](/images/2025-12-31-mobilehacking-2/image-20251231105932087.png)

<br>

adb install 명령을 이용해 adb와 연결된 디바이스 기기에 apk 파일을 설치해주었다.

![image-20251231110059277](/images/2025-12-31-mobilehacking-2/image-20251231110059277.png)

<br>

apk 파일을 열어보니 [GENERATE STRING] 버튼과 [HINT] 버튼이 있다. GENERATE STRING 버튼을 눌러보자.

![image-20251231110156113](/images/2025-12-31-mobilehacking-2/image-20251231110156113.png)

<br>

눌러보니 뭔가 알 수 없는 문자열이 생성된 것을 확인할 수 있다.

![image-20251231110256347](/images/2025-12-31-mobilehacking-2/image-20251231110256347.png)

<br>

AndroidManifest.xml 파일을 보았을 때 특별한 건 없고, 액티비티가 메인 한개만 존재한다는 것을 알 수 있다.

![image-20251231111029412](/images/2025-12-31-mobilehacking-2/image-20251231111029412.png)

<br>

메인 액티비티 소스 코드를 분석해보면 `babyanalysis` 라이브러리 파일을 로드하고 있다. 또한 `viewFindViewById` 에는 textView, `viewFindViewById2` 에는 GENERATE 버튼을, `viewFindViewById3` 에는 HINT 버튼을 의미한다. GENERATE 버튼을 누를 때에는 `onCreate$lambda$0()` 메소드가 실행되고, HINT 버튼을 누를 때는 `onCreate$lambda$1()` 메소드가 실행된다.

```java
public final class MainActivity extends AppCompatActivity {
    private static final String TAG = "BabyAnalysis";
    private Button generateButton;
    private TextView resultTextView;

    public final native String generateString();

    static {
        System.loadLibrary("babyanalysis");
    }

    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(C1193R.layout.activity_main);
        View viewFindViewById = findViewById(C1193R.id.resultTextView);
        Intrinsics.checkNotNullExpressionValue(viewFindViewById, "findViewById(R.id.resultTextView)");
        this.resultTextView = (TextView) viewFindViewById;
        View viewFindViewById2 = findViewById(C1193R.id.generateButton);
        Intrinsics.checkNotNullExpressionValue(viewFindViewById2, "findViewById(R.id.generateButton)");
        Button button = (Button) viewFindViewById2;
        this.generateButton = button;
        if (button == null) {
            Intrinsics.throwUninitializedPropertyAccessException("generateButton");
            button = null;
        }
        button.setOnClickListener(new View.OnClickListener() { // from class: mobilehacking.kr.mobilebabyanalysis3.MainActivity$$ExternalSyntheticLambda0
            @Override // android.view.View.OnClickListener
            public final void onClick(View view) {
                MainActivity.onCreate$lambda$0(this.f$0, view);
            }
        });
        View viewFindViewById3 = findViewById(C1193R.id.hintButton);
        Intrinsics.checkNotNullExpressionValue(viewFindViewById3, "findViewById(R.id.hintButton)");
        ((Button) viewFindViewById3).setOnClickListener(new View.OnClickListener() { // from class: mobilehacking.kr.mobilebabyanalysis3.MainActivity$$ExternalSyntheticLambda1
            @Override // android.view.View.OnClickListener
            public final void onClick(View view) {
                MainActivity.onCreate$lambda$1(this.f$0, view);
            }
        });
    }
```

<br>

따라서 GENERATE 버튼을 눌렀을 때 실행되는 `onCreate$lambda$0` 메소드를 분석해보면, `strGenerateString` 변수에 `this$0.generateString()`의 반환값을 저장하고 있다. `generateString()` 메소드는 상단에서 native 메소드로 선언되어 있으므로, 실제 구현은 라이브러리 내부에 존재하며 Ghidra를 이용해 분석을 진행할 것이다. 또한 strGenerateString 값을 textView에 설정하여 화면에 출력하고, 로그를 통해서도 해당 문자열을 출력하고 있음을 확인할 수 있다.

```java
public final native String generateString();

    /* JADX INFO: Access modifiers changed from: private */
    public static final void onCreate$lambda$0(MainActivity this$0, View view) {
        Intrinsics.checkNotNullParameter(this$0, "this$0");
        String strGenerateString = this$0.generateString();
        TextView textView = this$0.resultTextView;
        if (textView == null) {
            Intrinsics.throwUninitializedPropertyAccessException("resultTextView");
            textView = null;
        }
        textView.setText(strGenerateString);
        Log.i(TAG, "Generated String: " + strGenerateString);
    }
```

<br>

이제 앱과 logcat에서도 grep으로 `GEnerated Stirng:` 문자열을 필터링 걸어 `strGenerateString` 값이 잘 나오는 걸 확인해보자.

<img src="/images/2025-12-31-mobilehacking-2/image-20251231234829329.png" alt="image-20251231234829329" style="zoom: 50%;" />

![image-20251231234949180](/images/2025-12-31-mobilehacking-2/image-20251231234949180.png)

<br>

native로 선언된 `generateString()` 메소드를 분석하기 위해서 라이브러리 파일을 봐야하는데, 아래 명령어를 통해서 ABI를 확인해보자. 

```
adb shell getprop ro.product.cpu.abi
```

![image-20251231235541762](/images/2025-12-31-mobilehacking-2/image-20251231235541762.png)

<br>

코드에서 보면 babyanalysis 라이브러리를 불러오고 있고, 각 ABI에 맞게 apk가 디컴파일된 폴더에서 lib -> arm64-v8a 폴더에 들어가면 하단처럼 libbabyanalysis.so 파일을 확인할 수 있다. 

```java
    static {
        System.loadLibrary("babyanalysis");
    }
```

![image-20251231235622474](/images/2025-12-31-mobilehacking-2/image-20251231235622474.png)

<br>

이제 Ghidra를 열어서 Import file로 so파일을 불러온 후 Symbol Tree 부분의 Filter에서 우리가 찾고싶은 메소드인 generateString을 검색하면 빨간색 박스처럼 functions 부분에 함수를 누르게 되면 오른쪽에 있는 화면처럼 디컴파일된 소스 코드를 볼 수 있다.

![image-20251231235257300](/images/2025-12-31-mobilehacking-2/image-20251231235257300.png)

<br>

`generateString`  함수를 분석해보면 중간에 `encrypt_flag()` 함수를 확인할 수 있다.

```c
undefined8 Java_mobilehacking_kr_mobilebabyanalysis3_MainActivity_generateString(long *param_1)

{
  void *pvVar1;
  long lVar2;
  undefined8 uVar3;
  byte local_50 [16];
  void *local_40;
  long local_38;
  
  lVar2 = tpidr_el0;
  local_38 = *(long *)(lVar2 + 0x28);
  encrypt_flag();
  pvVar1 = (void *)((ulong)local_50 | 1);
  if ((local_50[0] & 1) != 0) {
    pvVar1 = local_40;
  }
                      /* try { // try from 0011e2ec to 0011e2f3 has its CatchHandler @ 0011e334 */
  uVar3 = (**(code **)(*param_1 + 0x538))(param_1,pvVar1);
  if ((local_50[0] & 1) != 0) {
    operator.delete(local_40);
  }
  if (*(long *)(lVar2 + 0x28) == local_38) {
    return uVar3;
  }
                      /* WARNING: Subroutine does not return */
  __stack_chk_fail();
}

```

<br>

ecnrypt_flag() 함수를 더블클릭하면 분석할 수 있으며, 먼저 lVar9라는 변수에 10진수 36값을 넣고, pbVar6 변수에 DAT_001140e0 주소 값에서 36바이트를 읽어오고 1을 더한 값과 XOR연산을 수행하여 local_50 변수에 저장한다. 이 값은 local_70이랑 local_88에도 동일한 값으로 저장되는 것을 알 수 있다.

```c
  while (lVar9 != 0x24) {
    pbVar6 = &DAT_001140e0 + lVar9;
    lVar9 = lVar9 + 1;
                      /* try { // try from 0011e024 to 0011e02b has its CatchHandler @ 0011e280 */
    std::__ndk1::basic_string<>::push_back((basic_string<> *)local_50,*pbVar6 ^ (byte)lVar9);
  }
                      /* try { // try from 0011e030 to 0011e03b has its CatchHandler @ 0011e270 */
  std::__ndk1::basic_string<>::basic_string(local_70,(basic_string *)local_50);
                      /* try { // try from 0011e03c to 0011e04b has its CatchHandler @ 0011e250 */
  std::__ndk1::basic_string<>::basic_string(local_88,(basic_string *)local_70);
```

<br>

DAT_001140e0 값도 알기 위해서 더블클릭하여 눌러보면 67h는 0x67로 알파벳 g를 의미하고, 6Eh는 n을 의미한다. 36바이트는 59h까지 읽으면된다.

![image-20260101004445349](/images/2025-12-31-mobilehacking-2/image-20260101004445349.png)

<br>

따라서 36바이트와 해당 값의 1을 더한 값과 XOR 연산을 수행하는 스크립트를 작성하면 flag 값을 얻을 수 있다!

```python
local_50 = [
  0x67, 0x6e, 0x62, 0x63, 0x7e, 0x74, 0x62, 0x7e, 0x6c, 0x78, 0x78,
  0x69, 0x52, 0x7a, 0x67, 0x75, 0x4e, 0x7c, 0x72, 0x60, 0x7c, 0x60,
  0x72, 0x47, 0x7a, 0x75, 0x7f, 0x79, 0x42, 0x7f, 0x71, 0x44, 0x7e,
  0x4f, 0x46, 0x59
]

data = ''

for i in range(0, len(local_50)):
    local_50[i] = local_50[i] ^ (i+1)
    data += chr(local_50[i])

print(data)
```

![image-20260101012814347](/images/2025-12-31-mobilehacking-2/image-20260101012814347.png)
