---
layout: single
title: "[Dreamhack] Flying Chars write-up"
date: 2024-02-03 00:02 +0900
categories: 
    - WEB-write-up
tag: web
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

날아다니는 글자들을 멈춰서 전체 문자열을 알아내세요! 플래그 형식은 DH{전체 문자열} 입니다.  

❗첨부파일을 제공하지 않는 문제입니다.  

❗플래그에 포함된 알파벳 중 `x`, `s`, `o`는 모두 소문자입니다.   

❗플래그에 포함된 알파벳 중 `C`는 모두 대문자입니다.  



<br>

## 풀이

문제 사이트로 접속하게 되면 다음과 같이 알파벳들이 전부 날아다닌다.

![image-20240203225440740](/images/2024-02-03-FlyingChars/image-20240203225440740.png)

<br>

이런 문제는 대부분 개발자 도구의 Console 창으로 해결할 수 있다. 먼저 [f12] 단축키를 눌러 개발자 도구에 들어간 후 날아다니는 코드를 분석해야 한다.

<br>

이 코드가 날아다니는 소스 코드는 아래와 같다. 

```javascript
<script type="text/javascript">
    const img_files = ["/static/images/10.png", "/static/images/17.png", "/static/images/13.png", "/static/images/7.png","/static/images/16.png", "/static/images/8.png", "/static/images/14.png", "/static/images/2.png", "/static/images/9.png", "/static/images/5.png", "/static/images/11.png", "/static/images/6.png", "/static/images/12.png", "/static/images/3.png", "/static/images/0.png", "/static/images/19.png", "/static/images/4.png", "/static/images/15.png", "/static/images/18.png", "/static/images/1.png"];
    var imgs = [];
    for (var i = 0; i < img_files.length; i++){
      imgs[i] = document.createElement('img');
      imgs[i].src = img_files[i]; 
      imgs[i].style.display = 'block';
      imgs[i].style.width = '10px';
      imgs[i].style.height = '10px';
      document.getElementById('box').appendChild(imgs[i]);
    }

    const max_pos = self.innerWidth;
    function anim(elem, pos, dis){
      function move() {
        pos += dis;
        if (pos > max_pos) {
          pos = 0;
        }
        elem.style.transform = `translateX(${pos}px)`;
        requestAnimationFrame(move);
      }
      move();
    }

    for(var i = 0; i < 20; i++){
      anim(imgs[i], 0, Math.random()*60+20);
    }
  </script>
```

<br>

`anim` 이라는 함수를 통해서 이러한 글자들이 날아다니는 것을 확인할 수 있다. 이 함수는 `elem, pos, dis` 를 인자로 사용하고 있는데 아래에 살펴보면 `elem` 자리에는 flag 글자들로 설정되어 있고, 2번은 0, 3번째는 랜덤함수를 사용해 이를 0으로 만들어주면 될 것 같다.

```javascript
function anim(elem, pos, dis){
      function move() {
        pos += dis;
        if (pos > max_pos) {
          pos = 0;
        }
        elem.style.transform = `translateX(${pos}px)`;
        requestAnimationFrame(move);
      }
      move();
    }

    for(var i = 0; i < 20; i++){
      anim(imgs[i], 0, Math.random()*60+20);
    }
```

<br>

바로 [Console] 탭으로 들어가서 아래와 같은 코드를 붙여넣어 주면 아래와 같이 flag 글자들이 한 줄로 쭉 멈춰지게 된다.

```javascript
for(var i = 0; i < 20; i++){
      anim(imgs[i], 0, 0);
}
```

![image-20240203230001393](/images/2024-02-03-FlyingChars/image-20240203230001393.png)

<br>

flag의 형식은 `DH{}` 이고 위의 알파벳을 옮겨적어주면 flag를 획득할 수 있다.