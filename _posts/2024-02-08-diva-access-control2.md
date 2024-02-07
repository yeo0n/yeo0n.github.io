---
layout: single
title: "[DIVA] Access Control Issues - Part 2"
date: 2024-02-08 03:13 +0900
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

## 액세스 제어 문제

액세스 제어 문제의 Part2로 넘어가면 등록하고 Tveeter API 자격 증명을 볼 수 있는 핀을 얻거나 핀이 이미 있는 경우 볼 수 있는 두 가지 옵션이 존재한다. 이 챌린지는 등록하지 않고 API 자격 증명에 액세스하는 것이다.

![image-20240208031431579](/images/2024-02-08-diva-access-control2/image-20240208031431579.png)

<br>
jadx 도구를 사용해 AndroidManifest.xml 파일을 분석해보면 전처럼 intent-filter 태그 안에 액션이 존재한다면 외부에서 접근이 가능하다. 따라서 `am start` 명령을 사용해서 액티비티를 실행할 수 있는 것이다.

![image-20240208031735560](/images/2024-02-08-diva-access-control2/image-20240208031735560.png)

<br>

아래와 같이 adb 쉘에 접속한 후, `am start jakhar.aseem.diva/.APICreds2Activity` 명령을 이용해서 액티비티를 실행하는데 `/`로 패키지와 액티비티를 구분해서 적어주면 아래 핀을 요구하는 액티비티에 접속한 것을 확인할 수 있다.

![image-20240208031955847](/images/2024-02-08-diva-access-control2/image-20240208031955847.png)

![image-20240208032003565](/images/2024-02-08-diva-access-control2/image-20240208032003565.png)

<br>

이 액티비티에서는 핀 번호를 검증하고 있어 아무 값이나 입력해보면 실패한다.

![image-20240208032418590](/images/2024-02-08-diva-access-control2/image-20240208032418590.png)

<br>
이제 이 핀 번호를 우회할 방법을 살펴보아야 한다. 분석해보면 `aci2rbregnow`의 값이 체크되어 있으면 `chk_pin`이 true가 된다. 또한 `setAction`을 통해 VIEW_CREDS2의 액티비티로 chk_pin 값을 전달하고 있다.

![image-20240208032244292](/images/2024-02-08-diva-access-control2/image-20240208032244292.png)

<br>

`chk_pin` 변수를 더블클릭하면 resources.arsc 파일에 아래처럼 표시되어 있다.

![image-20240208033714889](/images/2024-02-08-diva-access-control2/image-20240208033714889.png)

<br>

jadx도구로  ctrl+shift+f 단축키를 이용해 리소스를 검색하면 chk_pin이 string태그를 이용해 `check_pin`으로 설정되어 있는 것을 볼 수 있다.

![image-20240208033944029](/images/2024-02-08-diva-access-control2/image-20240208033944029.png)

<br>

APICreds2Activity 클래스를 살펴보면 `bcheck` 변수를 이용해 `chk_pin`이 false이면 API 크리덴셜 정보를 보여주는 걸 알 수 있다.

![image-20240208032306862](/images/2024-02-08-diva-access-control2/image-20240208032306862.png)

<br>
따라서 아래와 같이 `chk_pin`의 값은 실제로 `check_pin`이므로 adb의 am 명령을 이용해 `-n`으로 APICreds2Activity의 액티비티를 명시적으로 지정해주고, `--ez` 옵션을 사용해 `check_pin`의 키를 지정해주고 value를 `false`로 지정해주면 된다. 여기서 `-n`옵션이 없으면 `am start` 명령은 암시적 인텐트를 사용하게 되어 `--ez` 옵션을 사용할 수 없다. 

![image-20240208034929864](/images/2024-02-08-diva-access-control2/image-20240208034929864.png)

![image-20240208034937022](/images/2024-02-08-diva-access-control2/image-20240208034937022.png)

<br>

### 대응 방안

1. 보안 기본 구성 검토
   - 기본 설정 및 구성이 올바르게 보안되어 있는지 확인하고 중요한 정보를 노출시키거나 불필요한 사용 권한을 제공하지 않도록 한다.
2. 자격 증명 및 권한 검토
   - 하드코드화된 기본 자격 증명을 사용하지 말고, 지나치게 허용되는 권한을 가진 응용 프로그램 파일을 저장하지 않는다. 응용 프로그램의 적절한 작동을 위해 필요한 권한만 요청한다.
3. 보안 네트워크 구성 검토
   - 클리어 텍스트 트래픽을 허용하지 않으며 가능한 경우 인증서를 고정하여 사용한다.
4. 디버깅 비활성화
   - 프로덕션으로 출시할 버전의 앱은 디버깅 기능을 비활성화한다.
5. 백업 모드 비활성화
   - 안드로이드 기기에서 백업 모드를 비활성화해 기기 백업에 앱 데이터가 포함되지 않도록 한다.