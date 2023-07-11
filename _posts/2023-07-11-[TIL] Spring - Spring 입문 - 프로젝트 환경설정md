---
layout: "post"
title: "Spring 입문 1.프로젝트 환경설정"
subtitle: "Spring 입문 1.프로젝트 환경설정"
date: 2023-07-11
author: "raindragonn"
lang: ko
categories: [Spring]
tags:
  - Spring
  - Spring Boot
  - 입문
---
## Spring

### 기본 스프링 세팅 도움

[](https://start.spring.io/)

스프링 부트 기반으로 스프링 프로젝트를 만들어준다.

### Maven vs Gradle

- 라이브러리 저장소 및 빌드 라이프사이클 관리
- 과거에는 Maven을 이용하지만 최근에는 Gradle로 넘어가는 추세

### Spring Boot

- snapshot은 정식 버전은 아님
- 3.0 부터는 **Java 17부터 지원**된다.

### Dependencies

- Spring Web
- Thymeleaf
    - 여러가지 사용되는 html을 만들어주는 템플릿 엔진중 하나

`@SpringBootApplication` 

- 내장된 톰캣 웹서버를 자체적으로 띄움

## gradle build vs intellij idea build

- 증분빌드의 차이(변경된 부분만 빌드를 하는 방식.)
- intellij idea build는 증분빌드를 지원함.
- 실제 빌드하고 실행(결과물을 확인)하기 위해서는 gradle을 이용하는 편이 좋다.
- 빠른 빌드(테스트)를 위해서는 IDEA를 이용하는 편이 좋다.
- 차이점으로 빌드의 결과물의 위치가 다르다.
    - gradle build의 경우 build 폴더로 결과물이 나오며
    - intellij idea build의 경우 out 폴더로 결과물이 나온다.

## 라이브러리 살펴보기 with gradle

- gradle은 의존관계를 관리해준다.
    - 하나의 라이브러리를 추가하더라도 해당 라이브러리가 의존(사용)하는 라이브러리를 자동적으로 추가해준다.
    - gradle 탭에서 종속 관계를 확인할 수 있다.
- spring-boot-start-web에 종속된 라이브러리
    - spring-boot-starter-tomcat → 웹서버
    - spring-boot-starter-logging
        - slf4j → 로그 인터페이스
        - logback → 실제 어떤 구현체로 로그를 출력할까
    - junit → 테스트 라이브러리

## View 환경설정

- resources/static/index.html
- 스프링 부트가 제공하는 Welcome Page 기능
- `thymeleaf` 템플릿 엔진을 사용.
    - thymeleaf 공식 사이트: https://www.thymeleaf.org/
    - 스프링 공식 튜토리얼: https://spring.io/guides/gs/serving-web-content/
    - 스프링부트 메뉴얼: https://docs.spring.io/spring-boot/docs/2.3.1.RELEASE/reference/html/spring-boot-features.html#boot-features-spring-mvc-template-engines
- 웹 어플리케이션의 첫 진입점 → Controller
    - @Controller
    - @GetMapping
        - http method 의 GET 을 의미
        - 파라미터를 통해 진입점을 제공한다.
        - return 값을 통해 뷰리졸버가 Resources의 뷰를 전달(tymeleaf)

![스크린샷 2023-07-11 17.27.49.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6ee55198-c960-4d6c-88e6-9418b427f693/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-07-11_17.27.49.png)

## 빌드하고 실행하기

```kotlin
//mac 기준

// 빌드하기
./gradlew build

// 빌드 후에는 build/libs에 jar 파일이 생성됨
// 실행하는 방법
java -jar ~~.jar

// 빌드에 문제가 있는경우 빌드 지우는법
./gradlew clean build
```

## 팁

- 직접 확인(println과 같은)하는것이 아닌 저장되는 로그로 관리해야함.