---
layout: single
title: "[wargame.kr] login filtering write-up"
date: 2024-02-04 15:43 +0900
categories: 
    - WEB-write-up
tag: sqli
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

```text
I have accounts. but, it's blocked.
can you login bypass filtering?
```

<br>

## 풀이

문제에 접속하면 다음과 같이 로그인 화면이 나온다.

![image-20240204154512268](/images/2024-02-04-login-filtering/image-20240204154512268.png)

<br>

### 소스 코드

```php
<?php

if (isset($_GET['view-source'])) {
    show_source(__FILE__);
    exit();
}

/*
create table user(
 idx int auto_increment primary key,
 id char(32),
 ps char(32)
);
*/

 if(isset($_POST['id']) && isset($_POST['ps'])){
  include("./lib.php"); # include for $FLAG, $DB_username, $DB_password.

  $conn = mysqli_connect("localhost", $DB_username, $DB_password, "login_filtering");
  mysqli_query($conn, "set names utf8");

  $id = mysqli_real_escape_string($conn, trim($_POST['id']));
  $ps = mysqli_real_escape_string($conn, trim($_POST['ps']));

  $row=mysqli_fetch_array(mysqli_query($conn, "select * from user where id='$id' and ps=md5('$ps')"));
  if(isset($row['id'])){
   if($id=='guest' || $id=='blueh4g'){
    echo "your account is blocked";
   }else{
    echo "login ok"."<br />";
    echo "FLAG : ".$FLAG;
   }
  }else{
   echo "wrong..";
  }
 }
?>
<!DOCTYPE html>
<style>
 * {margin:0; padding:0;}
 body {background-color:#ddd;}
 #mdiv {width:200px; text-align:center; margin:50px auto;}
 input[type=text],input[type=[password] {width:100px;}
 td {text-align:center;}
</style>
<body>
<form method="post" action="./">
<div id="mdiv">
<table>
<tr><td>ID</td><td><input type="text" name="id" /></td></tr>
<tr><td>PW</td><td><input type="password" name="ps" /></td></tr>
<tr><td colspan="2"><input type="submit" value="login" /></td></tr>
</table>
 <div><a href='?view-source'>get source</a></div>
</form>
</div>
</body>
<!--

you have blocked accounts.

guest / guest
blueh4g / blueh4g1234ps

-->
```

<br>

소스 코드의 제일 아랫 부분에는 guest와 blueh4g의 계정 id와 pw가 적혀있다.

```text
<!--

you have blocked accounts.

guest / guest
blueh4g / blueh4g1234ps

-->
```

<br>

### FLAG 조건

id와 ps를 POST 메소드로 전송하여 `mysqli_real_escape_string` 함수를 사용하여 이스케이프 처리를 수행하였다. 요청되는 쿼리를 살펴보면 `select * from user where id='$id' and ps=md5('$ps')"));` 를 수행하고, 결과의 id가 `guest` 이거나 `blueh4g`이면 flag를 출력한다.

```php
 if(isset($_POST['id']) && isset($_POST['ps'])){
  include("./lib.php"); # include for $FLAG, $DB_username, $DB_password.

  $conn = mysqli_connect("localhost", $DB_username, $DB_password, "login_filtering");
  mysqli_query($conn, "set names utf8");

  $id = mysqli_real_escape_string($conn, trim($_POST['id']));
  $ps = mysqli_real_escape_string($conn, trim($_POST['ps']));

  $row=mysqli_fetch_array(mysqli_query($conn, "select * from user where id='$id' and ps=md5('$ps')"));
  if(isset($row['id'])){
   if($id=='guest' || $id=='blueh4g'){
    echo "your account is blocked";
   }else{
    echo "login ok"."<br />";
    echo "FLAG : ".$FLAG;
   }
  }else{
   echo "wrong..";
  }
 }
?>
```

<br>
먼저 `mysqli_real_escape_string` 함수의 이스케이프 문자 목록은 아래와 같다. 이 함수는 멀티바이트를 사용하는 언어셋 환경에서는 백슬래시(`\`) 앞에 %a1 ~ %fe 와 같은 값을 입력하면 `%a1\` 을 하나의 문자로 취급한다. 하지만 이 소스 코드에서는 특별히 멀티바이트를 사용하지는 않는 것 같다.

```php
'   "   \   \x00(NUL)   \x1a(EOF)   \n   \r
```

<br>

이 문제에서는 요청되는 쿼리에 `mysqli_real_escape_string` 함수 말고 따로 필터링 되는 함수는 존재하지 않는다. 따라서 FLAG 조건을 보게 되면 id가 `guest`이거나 `blueh4g` 이면 blocked 문자열을 출력하게 되는데 `Guest`라면 어떻게 수행될까?

<br>

소스 코드 부분에선 소문자를 단정지어 `guest`와 `blueh4g`를 검사하지만 실제 요청되는 쿼리는 대소문자를 구별하지 않는다는 것이다. 따라서 `Guest`와 `guest`를 각각 id와 ps에 넣어주면 flag를 획득할 수 있다.

![image-20240204162838070](/images/2024-02-04-login-filtering/image-20240204162838070.png)
