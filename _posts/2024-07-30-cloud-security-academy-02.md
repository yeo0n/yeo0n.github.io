---
layout: single
title: AWS 클라우드 - 시큐리티아카데미 17일차
date: 2024-07-30 20:22 +0900
categories: 
    - CLOUD-study
#tag: CLOUD-study
typora-root-url: ../
toc: true
toc_sticky: true
toc_label: 목차
author_profile: true
comment: true
sidebar:
    nav: "docs"

---

## AWS 클라우드 - 시큐리티 아카데미 17일차

<br>

### Amazon CloudFront

- CDN(Content Delivery Network)
- 간단히 HTTP/HTTPS를 이용하여 S3 및 ELB, EC2, 외부서버 등을 캐시하고 빠른 속도로 컨ㅌ네츠를 전달하는 캐시서버
- 전 세계 각지에 퍼져 있는  Edge Location의 주변 Origin Server(EC2 등)의 컨텐츠를 Edge Location에 캐싱하고 각 Edge Location간 공유를 통해 컨텐츠를 전달
- Distribution은 Edge Location의 집합을 의미
- 각 Edge Location간에는 아마존의 백본 네트워크를 통하기 때문에 매우 빠른 속도로 전달 가능

<br>

### Amazon VPC(Virtual Private Cloud)

- VPC란?
  - AWS 계정 전용 가상 네트워크 서비스
  - VPC 내에서 각종 리소스(EC2, RDS, ELB 등)을 시작할 수 있으며 다른 가상 네트워크와 논리적으로 분리되어 있음
  - S3, CloudFront 등은 VPC 서비스로 VPC 내에서 생성되지 않음
  - 각 Region별로  VPC가 다수 존재할 수 있음
  - VPC는 하나의 사설 IP 대역을 보유하고, 서브넷을 생성하며 사설 IP 대역 일부를 나누어 줄 수 있음
  - 허용된 IP 블록 크기는 /16(IP 65536개) ~ /28(IP 16개)
  - 권고하는 VPC CIDR 블록(사설 IP 대역과 동일함)
    - 10.0.0.0 ~ 10.255.255.255(10.0.0.0/8)
    - 172.16.0.0 ~ 172.31.255.255(172.16.0.0/12)
    - 192.168.0.0 ~ 192.168.255.255(192.168.0.0/16)
- Subnet
  - VPC 내 생성된 분리된 네트워크로 하나의 서브넷은 하나의 AZ엥 ㅕㄴ결
  - VPC가 가지고 있는 사설  IP 범위 내에서 서브넷을 쪼개어 사용 가능
  - 실질적으로 리소스들은 이 서브넷에서 생성이 되며 사설 IP를 기본적으로 할당받고 필요에 따라 공인 IP를 할당받음 
  - 하나의 서브넷은 하나의 라우팅 테이블과 하나의 NACL을 가짐 
  - 각 서브넷의 CIDR 블록에서 4개의 IP 주소와 마지막 IP 주소는 예약 주소로 사용자가 사용할 수 없음, 예를 들어 주소가 172.16.1.0/24일 경우
    - 172.16.1.0: 네트워크 주소
    - 172.16.1.1: VPC Router용 예약 주소
    - 172.16.1.2: DNS 서버의 IP 주소
    - 172.16.1.3: 향후 사용할 예약 주소 
    - 172.16.1.255: 네트워크 브로드캐스트 주소

<br>

### 클라우드 실습 

<br>

#### MFA 설정 및 EC2 생성

- MFA 추가 

  - IAM > MFA
  - MFA 디바이스 관리 > 가상 MFA 디바이스 
  - 구글 OTP 앱 설치
  - QR 코드 

- instnace 생성

  - vpc 및 네트워크 설정

  - 보안그룹 설정

  - amazon linux라면 후에 ssh로 접속할 때 ec2-user 이름으로 로그인하기(Ubuntu는 사용자이름 ubuntu)

  - windows는 putty로 접속 가능

    - putty 설정

      \- category > terminal > keyboard >backspace key

      => control+H

      \- category > terminal > bell > none

      \- category > window > appearance > font setting > 14

      \- putty key generator > conversions > import key > save private key통해 pem을 ppk로 변경 후 저장

      \- category > connection > SSH > Auth > credential > AWS.ppk

      \- category > session > saved sessions > AWS(또는 oregon) > save

<br>

#### VPC 설정

- 이를 하는 이유는 조금 더 보안적으로 안전하고 AWS 리소스간 허용을 최소화하고 그룹별로 손쉽게 네트워크를 구성하기 위해 사용한다. private 환경에 접속하기 위해서 동일한 AZ 가용 공간 내에서 두 개의 인스턴스가 하나는 퍼블릭, 하나는 프라이빗으로 존재해서 인터넷 게이트웨이랑 EIP, NATGW, 서브넷 등을 설정해줌으로써 퍼블릭 서버로 키페어를 scp 명령어를 이용해 보낸 후 이 키페어를 이용해 프라이빗서버로 ssh 접속할 수 있음 

  - \## VPC

    -vpc 생성

    ​	- vpc만

    ​	- 이름: example

    ​	- cidr: 10.0.0.0/16

    \- subnet 생성

    ​	- 이름: example-public-1a

    ​	- 가용영역: ap-northeast-2a

    ​	- cidr: 10.0.1.0/24

    

    ​	- 이름: example-private-2a

    ​	- 가용영역: ap-northeast-2a

    ​	- cidr: 10.0.2.0/24

    

    ​	- 이름: example-public-3c

    ​	- 가용영역: ap-northeast-2c

    ​	- cidr: 10.0.3.0/24

    

    ​	- 이름: example-private-4c

    ​	- 가용영역: ap-northeast-2c

    ​	- cidr: 10.0.4.0/24

    

    \- internet gateway 생성

    ​	- 이름: example-IGW

    ​	- vpc에 연결(example vpc)

    

    \- EIP 생성

    ​	- 이름: example-NATGW-EIP

    

    \- NATGW 생성

    ​	- 이름: example-NATGW

    ​	- subnet: example-public-1a

    ​	- EIP: 생성된 ip 선택

    

    \- Routing Table

    ​	- 라우팅

    ​	- 이름: example-public-RT

    ​	- 라우팅편집: 0.0.0.0/0, example-IGW

    ​	- 서브넷연결

    ​	- 퍼블릭 서브넷 선택

    

    ​	- 라우팅

    ​	- 이름: example-private-RT

    ​	- 라우팅편집: 0.0.0.0/0, example-NATGW

    ​	- 서브넷연결

    ​	- 프라이빗 서브넷 선택

    

    \- EC2 생성

    ​	- bastion 생성(public1a)

    ​	- private_EC2 생성(private4c)

    

    \- bastion 접속

    \- private_ec2 접속

    ​	- (window_cmd)

    ​	- cd Downloads

    ​	- scp -i AWS_2.pem AWS_2.pem ec2-user@{퍼블릭IP}:/home/ec2-user

    ​	- chmod 600 AWS_2.pem

    ​	- ssh -i AWS_2.pem {프라이빗IP}