---
layout: single
title: "[wargame.kr] command-injection-chatgpt write-up"
date: 2024-02-04 18:03 +0900
categories: 
    - WEB-write-up
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

## 문제 설명

특정 Host에 ping 패킷을 보내는 서비스입니다.  

Command Injection을 통해 플래그를 획득하세요. 플래그는 `flag.py`에 있습니다.  

[chatGPT](https://chat.openai.com/)와 함께 풀어보세요!  

<br>

## 풀이

문제에 먼저 접속하면 placeholder로 Host에 8.8.8.8을 입력하도록 되어있다.

![image-20240204180423291](/images/2024-02-04-command-injection-chatgpt/image-20240204180423291.png)

<br>

Ping! 버튼을 눌러보면 다음과 같이 ping 명령 결과가 잘 출력되는 것을 볼 수 있다.

![image-20240204180501680](/images/2024-02-04-command-injection-chatgpt/image-20240204180501680.png)

<br>

Command Injection은 `;` 문자나 `&` 문자 등을 통해서 두 개의 명령을 동시에 실행할 수 있는 취약점이다. 따라서 `8.8.8.8; ls` 를 입력하면 8.8.8.8에 대한 핑 명령어가 수행되고 다음으로 ls 명령어가 수행된다.

![image-20240204180632481](/images/2024-02-04-command-injection-chatgpt/image-20240204180632481.png)

<br>

`8.8.8.8; cat flag.txt` 명령어를 입력하면 flag를 확인할 수 있다.