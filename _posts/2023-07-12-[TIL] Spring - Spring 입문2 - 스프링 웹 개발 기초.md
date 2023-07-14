---
layout: "post"
title: "Spring 입문 2.스프링 웹 개발 기초"
subtitle: "Spring 입문 2.스프링 웹 개발 기초"
date: 2023-07-12
author: "raindragonn"
lang: ko
categories: [Spring]
tags:
  - Spring
  - Spring Boot
  - 입문
---
## 스프링 웹 개발 기초

참고 강의:https://www.inflearn.com/roadmaps/373 

### 웹을 개발한다의 간략한 설명

1. 정적 컨텐츠
    - 웰컴 페이지와 같은 서버작업 없이 웹브라우저에 파일 그대로 보여줌
2. MVC와 템플릿 엔진
    - JSP, PHP와 같은 템플릿 엔진
    - 서버를 통해 html을 동적으로 가공해 웹브라우저에 보여줌
    - Model, View, Controller
3. API
    - json 데이터 구조 포맷을 통해 클라이언트에게 전달
    - 서버끼리 통신할 때사용

### 정적 컨텐츠

- `/static` 아래 내용을 통해 보여줌
    - ex) hello-static.html ⇒ [localhost:8080/hello-static.html](http://localhost:8080/hello-static.html)

간단한 원리

![Untitled](/assets/images/post/2023-07-12-1.png)

1. 요청에따른 관련 컨트롤러를 찾아봄
2. resources를 통해 정적 컨텐츠를 찾아 보여줌

### MVC와 템플릿 엔진

과거 JSP 에서는 View에서 모든걸 다했음,  현재는 MVC로 각 책임을 분리하고 템플릿엔진을 통해 가공된 View를 제공한다.

View - 화면을 그리는 것에 집중한다.

Model - 데이터에 집중한다.

Controller - 비즈니스 로직에 집중한다.

![Untitled](/assets/images/post/2023-07-12-2.png)

### API

- View가 아닌 데이터를 반환
- `@ResponseBody`
    - http 통신 프로토콜의 body 부분에 추가하겠다.
    - Spring에서 해당 어노테이션을 사용해서 객체를 반환하는 경우 기본적으로 json으로 반환한다.
        - `HttpMessageConverter` 를 통함.
        - 객체의 경우 `MappingJackson2HttpMessageConverter`, 단순 문자의 경우 `StringHttpMessageConverter`로 처리한다.
- 과거에는 xml로 반환했지만 현재는 일반적으로 json 형태로 반환
![Untitled](/assets/images/post/2023-07-12-3.png)