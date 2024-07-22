---
layout: single
title: "Damn Vulnerable-Bank 설치 / 루팅,frida 탐지 우회"
date: 2024-07-17 22:05 +0900
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

## Damn Vulnerable-Bank 설치 / 루팅,frida 탐지 우회

<br>

설치하기 전에 먼저 안드로이드 에뮬레이터를 먼저 생성해주어야 한다. 윈도우를 사용하면 Nox 에뮬레이터를 사용할 수 있고 Mac을 이용한다면 안드로이드 스튜디오를 이용해 에뮬레이터를 생성해주고 루팅을 해주자. macOS의 다른 안드로이드 에뮬레이터로 mumuplayer, Genymotion 정도가 있는 것 같으니 확인해보는 것도 좋을듯하다.

<br>
다음으로 설치하기 전에 도구들을 설치해주어야 하는데 여기서 필요한 도구는 다음과 같다. 여기서 일단 나는 apkx는 jadx-gui로 대체할 예정이다. apkx 도구는 jd-gui를 이용하기 위해 dex파일을 dex2jar도구로 jar로 변환해주는데 쉽게 도와주는 도구인 것 같다. 

- adb
- frida
- apkx
- apktool
- ghidra
- burp suite

<br>

https://github.com/rewanthtammana/Damn-Vulnerable-Bank/raw/master/dvba.apk 링크를 통해서 apk 파일을 다운받은 후 adb install 명령을 이용해 설치를 진행

<img src="/images/2024-07-17-Damn-Vulnerable-Bank1/image-20240717221620526.png" alt="image-20240717221620526" style="zoom:50%;" />

<br>

그리고 에뮬레이터에서 앱을 실행시켜 보면 Rooted, frida is running 이라는 문자열이 나오면서 실행이 되지 않는다. 

<img src="/images/2024-07-17-Damn-Vulnerable-Bank1/image-20240717221721438.png" alt="image-20240717221721438" style="zoom:50%;" />

<img src="/images/2024-07-17-Damn-Vulnerable-Bank1/image-20240717221818741.png" alt="image-20240717221818741" style="zoom:50%;" />

<br>

apktool 도구를 이용하여 dvba.apk 파일이 존재하는 경로에서 `apktool d dvba.apk` 명령을 입력하여 디컴파일을 진행해주자.

![image-20240717222127413](/images/2024-07-17-Damn-Vulnerable-Bank1/image-20240717222127413.png)

![image-20240717222259035](/images/2024-07-17-Damn-Vulnerable-Bank1/image-20240717222259035.png)

<br>
jadx-gui 도구를 이용해서 아래와 같이 dvba.apk 파일을 열어주어도 소스 코드를 확인할 수 있다.

![image-20240717224115215](/images/2024-07-17-Damn-Vulnerable-Bank1/image-20240717224115215.png)

<br>
먼저 apk 파일이 Rooted와 Frida is running 문자열이 나오면서 실행되지 않으니 이를 우회하여야 한다. Rooted 문자열을 jadx-gui을 이용해 검색해주면 아래와 같이 관련된 소스 코드를 확인할 수 있다.

![image-20240717224456528](/images/2024-07-17-Damn-Vulnerable-Bank1/image-20240717224456528.png)

<br>

해당 소스 코드를 살펴보면 먼저 가장 위에 패키지 경로를 확인할 수 있다. `com.app.damnvulnerablebank` 안의 Dashboard인 것을 알 수 있다.

![스크린샷 2024-07-17 오후 10.56.10](/images/2024-07-17-Damn-Vulnerable-Bank1/스크린샷 2024-07-17 오후 10.56.10.png)

<br>

frida를 먼저 사용하기 위해서 앱을 종료해주어야 한다. `frida-ps -Uai`로 현재 실행중인 앱 및 프로세스를 확인할 수 있다. 그리고 `frida-kill -U 프로세스명` 명령을 통해 앱을 종료할 수 있다.

![image-20240717230538042](/images/2024-07-17-Damn-Vulnerable-Bank1/image-20240717230538042.png)

![image-20240717230800307](/images/2024-07-17-Damn-Vulnerable-Bank1/image-20240717230800307.png)

<br>
다시 Rooted 문자열이 나온 소스 코드를 살펴보면 아래와 같고,`a.a.a.a.a.R()` 을 통해 True인지 검사하기 때문에 `R()` 메서드를 확인해보면 아래와 같이 su 문자열 등을 검사하는 것을 알 수 있다. 주석으로 처리된 것은 디컴파일러가 메서드 구조를 나타내기 위해 표시한 것이라고 한다. ![image-20240717231353666](/images/2024-07-17-Damn-Vulnerable-Bank1/image-20240717231353666.png)

![image-20240717232352010](/images/2024-07-17-Damn-Vulnerable-Bank1/image-20240717232352010.png)

<br>

이제 frida를 이용해 R() 메서드를 false로 만들어주면 루팅 탐지를 우회할 수 있다. 아래와 같이 perform을 이용해 현재 스레드가 가상머신에 연결되어 있는지 확인하고 아래 function을 호출한다. 그리고 use 메서드를 이용해 a.a.a.a.a 패키지 경로의 R 메서드를 implementation을 이용해 재정의해주어 false 값을 주었다. 

![image-20240717232828556](/images/2024-07-17-Damn-Vulnerable-Bank1/image-20240717232828556.png)

<br>

그럼 아래와 같이 -U로 패키지 명을 지정해주고 -f로 패키지 전에 후킹 코드를 집어넣는다. 그리고 -l 옵션으로 후킹 소스를 넣어주면 아래와 같이 실행되면서 에뮬레이터에서 frida is running 문자열만 나오는 것을 확인할 수 있다.

![image-20240717234225981](/images/2024-07-17-Damn-Vulnerable-Bank1/image-20240717234225981.png)

<br>
frida 문자열을 검색하면 해당 탐지 코드를 쉽게 찾을 수 있는데 라이브러리를 로드하는 것을 알 수 있다. 이는 Ghidra로 분석해야 한다. 

![image-20240717233942945](/images/2024-07-17-Damn-Vulnerable-Bank1/image-20240717233942945.png)

<br>

먼저 macOS에서는 Ghirda가 설치된 폴더 경로 또는 /usr/local/bin에 실행 파일을 옮겨주고 GhidraRun 명령을 통해 실행한 한다. 다음으로 프로젝트를 생성해주고 import file 메뉴를 선택해 에뮬레이터 아키텍처에 맞는 so 파일을 업로드 해주는데 frida-check.so 를 분석하라고 이름까지 친절히 안내해준다. 이를 더블클릭하면 아래와 같은 화면이 나온다. 왼쪽 아래 filter를 통해 frida를 검색하면 해당 함수의 디컴파일 소스 코드를 오른쪽에서 볼 수 있다.

![image-20240717235339861](/images/2024-07-17-Damn-Vulnerable-Bank1/image-20240717235339861.png)

<br>

아래 소스 코드를 살펴보면 frida를 check하기 위해 포트를 검사하고 있는 것을 알 수 있다. frida의 기본 포트는 20742이다.

```c
void Java_com_app_damnvulnerablebank_FridaCheckJNI_fridaCheck(void)

{
  long lVar1;
  int __fd;
  int iVar2;
  uint uVar3;
  sockaddr local_38;
  long local_28;
  
  lVar1 = tpidr_el0;
  local_28 = *(long *)(lVar1 + 0x28);
  __fd = socket(2,1,0);
  if (-1 < __fd) {
    local_38.sa_family = 2;
    local_38.sa_data[0] = 'i';
    local_38.sa_data[1] = -0x5e;
    iVar2 = inet_pton(2,"127.0.0.1",(void *)((ulong)&local_38 | 4));
    if (0 < iVar2) {
      uVar3 = connect(__fd,&local_38,0x10);
      uVar3 = ~uVar3 >> 0x1f;
      goto LAB_001007ac;
    }
  }
  uVar3 = 0;
LAB_001007ac:
  if (*(long *)(lVar1 + 0x28) == local_28) {
    return;
  }
                    /* WARNING: Subroutine does not return */
  __stack_chk_fail(uVar3);
}
```

<br>

`adb forward tcp:1337 tcp:1337` 명령으로 1337포트를 포트포워딩 해준 후에 `adb forward --list` 명령을 통해 포트포워딩이 되었는지 확인할 수 있다.

<br>

frida 포트를 다른 포트로 바꾸기 위해서는 frida를 먼저 `ps-ef | grep frida` 명령으로 프로세스를 찾고 `kill -9`로 서버를 죽인 후 frida 서버를 다시 실행할 때 뒤에 `-l 0.0.0.0:1337` 을 입력하여 frida 서버가 1337로 동작하도록 변경해준다.

![image-20240718000634719](/images/2024-07-17-Damn-Vulnerable-Bank1/image-20240718000634719.png)

<br>

이제 다시 루트 탐지를 우회하는데 frida의 호스트를 127.0.0.1:1337로 설정해주고 후킹해주어야 한다. -H 옵션을 통해서 설정해줄 수 있다.

```bash
frida -H 127.0.0.1:1337 -f com.app.damnvulnerablebank -l scripts/script.js
```

![image-20240721213312435](/images/2024-07-17-Damn-Vulnerable-Bank1/image-20240721213312435.png)

<br>
그럼 다음과 같이 Damn Vulnerable Bank 앱이 잘 열리는 것을 확인할 수 있다.

![image-20240721213329952](/images/2024-07-17-Damn-Vulnerable-Bank1/image-20240721213329952.png)
