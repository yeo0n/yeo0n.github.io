---
layout: single
title: "[Dreamhack] Tomcat Manger write-up"
date: 2025-11-12 18:30 +0900
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

## [Dreamhack] Tomcat Manager

### 😁 문제 설명

- 드림이가 톰캣 서버로 개발을 시작하였습니다.
  서비스의 취약점을 찾아 플래그를 획득하세요.
  플래그는 `/flag` 경로에 있습니다.

<br>

### ✏️ 풀이

문제 주소에 접속하면 아래와 같이 Coming Soon.. 이미지가 나온다.

![image-20251112183117820](/images/2025-11-12-Tomcat-Manager/image-20251112183117820.png)

<br>

index.jsp의 소스 코드인데 file 파라미터로 working.png 이미지 파일을 불러오고 있다.

```jsp
<html>
<body>
    <center>
        <h2>Under Construction</h2>
        <p>Coming Soon...</p>
        <img src="./image.jsp?file=working.png"/>
    </center>
</body>
</html>
```

<br>

아래는 `image.jsp` 파일의 소스 코드이고, 살펴보면 _file 변수에 file 파라미터 값을 받아와 그대로 파일 경로에 붙여서 열고 있다. 이 과정에서 별다른 검증이 없다. 따라서  file 파라미터 값에 `../../` 를 활용해 file download 취약점을 활용할 수 있다.

```jsp
<%@ page trimDirectiveWhitespaces="true" %>
<%
String filepath = getServletContext().getRealPath("resources") + "/";
String _file = request.getParameter("file");

response.setContentType("image/jpeg");
try{
    java.io.FileInputStream fileInputStream = new java.io.FileInputStream(filepath + _file);
    int i;   
    while ((i = fileInputStream.read()) != -1) {  
        out.write(i);
    }   
    fileInputStream.close();
}catch(Exception e){
    response.sendError(404, "Not Found !" );
}
%>
```

<br>

문제 이름이 Tomcat Manager 이므로 `/manager/html` 경로에 접속해봤을 때 사용자 이름과 비밀번호를 입력하는 창이 나오는 것을 알 수 있다.

![image-20251112190006391](/images/2025-11-12-Tomcat-Manager/image-20251112190006391.png)

<br>

아래는 문제 파일에서 tomcat-users.xml 파일안의 내용인데 tomcat 유저의 패스워드가 secret으로 되어 있어 알 수 없다.

```xml
<user username="tomcat" password="[**SECRET**]" roles="manager-gui,manager-script,manager-jmx,manager-status,admin-gui,admin-script" />
```

<br>

이제 image.jsp 파일의 file 파라미터 취약점을 이용해 tomcat-users.xml을 열어보면 tomcat의 비밀번호를 알 수 있다. 문제의 dockerfile에서는 `tomcat-usesrs.xml` 파일을 `/usr/local/tomcat/conf/tomcat-users.xml` 경로에 저장되어있다.

```dockerfile
COPY tomcat-users.xml /usr/local/tomcat/conf/tomcat-users.xml
```

<br>

burp를 이용해서 확인해보면 tomcat의 비밀번호가 잘 출력되는 것을 확인할 수 있다.

![image-20251112190656045](/images/2025-11-12-Tomcat-Manager/image-20251112190656045.png)

<br>

`/manager/html` 경로에서는 아래 WAR file to deploy에서 war 파일 업로드 기능을 통해 웹쉘을 업로드할 수 있다. deploy를 하게 되면 위에 생성된 `/webshell` 경로가 deploy 된 것을 볼 수 있다.

![image-20251112192113833](/images/2025-11-12-Tomcat-Manager/image-20251112192113833.png)

<br>

`ls /` 명령어로 확인해보면, flag 파일이 있는 것을 확인할 수 있고 flag 파일을 실행하면 flag가 출력된다.

![image-20251112192035274](/images/2025-11-12-Tomcat-Manager/image-20251112192035274.png)
