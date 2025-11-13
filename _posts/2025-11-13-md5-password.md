---
layout: single
title: "[Dreamhack] md5 password write-up"
date: 2025-11-13 18:26 +0900
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

## [Dreamhack] md5 password

### 😁문제 설명

```text
md5('value', true);
```

<br>

### 풀이

문제에 접속하면 아래와 같이 패스워드 입력하는 폼이 존재한다.

![image-20251113185610694](/images/2025-11-13-md5-password/image-20251113185610694.png)

<br>

`get_source` 를 누르면 아래 소스 코드를 확인할 수 있는데, 문제 설명처럼 md5의 두 번째 인자를 true로 하여 admin의 패스워드인지 확인하고 있다.

```php+HTML
<?php
 if (isset($_GET['view-source'])) {
  show_source(__FILE__);
  exit();
 }

 if(isset($_POST['ps'])){
  sleep(1);
  include("./lib.php"); # include for $FLAG, $DB_username, $DB_password.
  $conn = mysqli_connect("localhost", $DB_username, $DB_password, "md5_password");
  /*
  
  create table admin_password(
   password char(64) unique
  );
  
  */

  $ps = mysqli_real_escape_string($conn, $_POST['ps']);
  $row=@mysqli_fetch_array(mysqli_query($conn, "select * from admin_password where password='".md5($ps,true)."'"));
  if(isset($row[0])){
   echo "hello admin!"."<br />";
   echo "FLAG : ".$FLAG;
  }else{
   echo "wrong..";
  }
 }
?>
<style>
 input[type=text] {width:200px;}
</style>
<br />
<br />
<form method="post" action="./index.php">
password : <input type="text" name="ps" /><input type="submit" value="login" />
</form>
<div><a href='?view-source'>get source</a></div>
```

<br>

php의 md5 함수에서는 두 번째 기본 인자값은 false인데, true로 변경할 경우 32자리의 16진수가 생성되는 것이 아니라 16자리의 바이너리가 생성된다.

<br>

따라서 바이너리 값에 특수문자가  포함되면 sql injection이 발생할 수 있다. 아래 값은 md5 true로 찾아보면 나오는 페이로드로 참고해보면 좋을 거 같다.

```
129581926211651571912466741651878684928
```

https://cvk.posthaven.com/sql-injection-with-raw-md5-hashes

<br>

위 값을 password에 넣으면 쉽게 flag를 얻을 수 있다.

![image-20251113191159966](/images/2025-11-13-md5-password/image-20251113191159966.png)