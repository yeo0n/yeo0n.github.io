---
layout: single
title: "[DIVA] Access Control Issues - Part 1"
date: 2024-02-08 01:59 +0900
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

이번에는 DIVA앱의 9번째에 있는 액세스 제어 문제를 실습할 것이다. 

![image-20240208020129589](/images/2024-02-08-diva-access-control1/image-20240208020129589.png)

<br>
9번째 액티비티에 접속하면 API의 계정 정보를 확인하라는 버튼이 존재하고 누르면 확인할 수 있다. 하지만 이 챌린지의 목적은 VIEW 버튼을 클릭하지 않고 API 자격 증명을 액세스하는 것이다.

![image-20240208020221329](/images/2024-02-08-diva-access-control1/image-20240208020221329.png)

![image-20240208020236839](/images/2024-02-08-diva-access-control1/image-20240208020236839.png)

<br>

이를 확인하기 위해서는 AndroidManifest.xml 파일을 확인해야 한다. 그럼 intent-filter 태그 안에 액션과 카테고리가 지정되어 있다. intent filter는 일반적으로 보호 메커니즘에 사용되지만 액션과 함께 사용하는 경우 공개적으로 내보내는 데에도 사용할 수 있다. 이렇게 될 경우 외부의 모든 애플리케이션이 이 활동을 사용하고 API 자격 증명을 볼 수 있다. 

![image-20240208022203204](/images/2024-02-08-diva-access-control1/image-20240208022203204.png)

<br>

그럼 ADB를 이용해서 이 액티비티를 읽어볼 수 있다. `/`를 이용해 안드로이드에서 패키지 이름과 액티비티 이름을 구분해줄 수 있다. `am`명령어를 사용할 수 있는데 `am start`와 뒤 경로를 통해 액티비티를 시작할 수 있다.  두번째 명령은 `-n` 옵션을 이용해 액티비티를 시작할 때 패키지 이름과 액티비티의 이름을 지정하는 옵션을 추가한 후 `-a` 옵션을 사용해 액티비티가 수행할 액션을 지정할 수 있다. 따라서 APICredsAcitivity 액티비티를 실행하면서 VIEW_CREDS 액션을 수행하도록 지정하는 것이다.

- ADB 쉘 외부에 있는 경우

  `adb shell am start jakhar.aseem.diva/APICredsActivity`

  ![image-20240208023119434](/images/2024-02-08-diva-access-control1/image-20240208023119434.png)

  `adb shell am start -n jakhar.aseem.diva/.APICredsActivity -a jakhar.aseem.diva.action.VIEW_CREDS`

  ![image-20240208023233731](/images/2024-02-08-diva-access-control1/image-20240208023233731.png)

- ADB 쉘 내부에 있는 경우

  `am start jakhar.aseem.diva/.APICredsActivity`

  ![image-20240208022845965](/images/2024-02-08-diva-access-control1/image-20240208022845965.png)

위 명령을 수행한 후에 에뮬레이터에 API Credentials가 보이는 액티비티가 잘 뜨는것을 확인할 수 있다.

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