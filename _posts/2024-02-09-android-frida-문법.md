---
layout: single
title: "Frida 기본 문법"
date: 2024-02-09 19:21 +0900
categories: 
    - Android
tag: 
#typora-root-url: ../
toc: true
toc_sticky: true
toc_label: 목차
author_profile: true
comment: true
sidebar:
    nav: "docs"
---

## Frida JavaScript API

### Java.perform(funtion)

- 현재 스레드가 가상머신에 연결되어 있는지 확인하고 function을 호출한다.

  ```javascript
  Java.perform(function() {
  	/*
  	...
  	*/
  })
  ```

<br>

### Java.use(calssName)

- 변수와 메소드에 접근할 수 있는 클래스 객체를 반환한다. (인스턴스가 아닌 클래스 객체를 반환하는 것)

- `.implementation`을 통해 클래스에 정의된 메소드를 재작성할 수 있다.

  ```javascript
  Java.perform(function() {
  	var myClass = Java.use(com.mypackage.name.class)
  	myClass.myMethod.implementation = function(param){
  	....
  	....
  	}
  })
  ```

<br>

### Java.choose(className, callbacks)

- 안드로이드 앱 내부의 인스턴스를 다루기 위한 것으로, 힙에서 인스턴스화 된 객체를 찾아 callback으로 넘겨준다.

- onMatch: 일치하는 인스턴스를 찾을 경우 호출한다.

- onComplete: 일치하는 것을 모두 찾고난 후에 호출한다. 

  ```javascript
  Java.perform(function() {
    Java.choose(com.mypackage.name.class, {
      "onMatch" : function(instance) {
        console.log(instance.toString())
      },
      "onComplete" : function() {}
    })
  })
  ```

<br>

### Java.enumerateLoadedClasses(callbacks)

- 로드된 모든 클래스를 열거한다.

  ```javascript
  Java.perform(function() {
  	Java.enumerateLoadedClasses({
  		"onMatch" : function(className) {
  			console.log(className)
  		},
  		"onComplete" : function() {}
  	})
  })
  ```

<br>

### setImmediate(function)

- Frida는 애뮬레이션이 느려질 경우, 시간초과 되어 연결을 자동으로 종료하는 경우가 있기 때문에 이를 막기 위해 사용한다. 

  ```javascript
  setImmediate(function() { // prevent timeout
    console.log("[*] starting script");
    
    Java.perform(function() {
      myClass = Java.use("com.package.name.class.name");
      myClass.implementation = function(v) {
        // do something
      }
    })
  })
  ```

<br>

### 오버로딩

> 오버로딩(Overloading)은 하나의 클래스 내에 동일한 이름의 메소드가 매개변수 정보를 달리하여 여러 개 존재하는 것을 말한다.

오버로딩을 구현하기 위해 `overload()`를 사용할 수 있다.

```javascript
myClass.myMethod.overload().implementation = function() {
  // 입력받는 인수가 없는 메소드
  // do something
})

myClass.myMethod.overload("[B", "[B").implementation = function(param1, param2) {
  // 두 개의 바이트 배열을 인수로 입력 받는 메소드
  // do something
})

myClass.myMethod.overload("android.context.Context", "boolean").implementation = function(param1, param2) {
  // 앱의 context와 Boolean 값을 인수로 입력받는 메소드
  // do something
})
```

