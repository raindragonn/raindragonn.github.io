---
layout: "post"
title: "Spring 핵심 원리 - 스프링 컨테이너와 스프링 빈"
subtitle: "Spring 핵심 원리 - 스프링 컨테이너와 스프링 빈"
date: 2023-08-07
author: "raindragonn"
lang: ko
categories: [Spring]
tags:
  - Spring
  - Spring Boot
  - 기본
---

## 스프링 컨테이너 생성

### 스프링 컨테이너가 생성되는 과정

```java
ApplicationContext applicationContext =
                new AnnotationConfigApplicationContext(AppConfig.class);
```

- `ApplicationContext` 를 스프링 컨테이너라 한다.
- `ApplicationContext` 는 인터페이스이다.
- 스프링 컨테이너는 XML을 기반으로 만들 수 있고, 애노테이션 기반의 자바 설정 클래스로 만들 수 있다.
- 직전에 AppConfig 를 사용했던 방식이 애노테이션 기반의 자바 설정 클래스로 스프링 컨테이너를 만든 것이다.
- 자바 설정 클래스를 기반으로 스프링 컨테이너( ApplicationContext )를 만들어보자.
    - `new AnnotationConfigApplicationContext(AppConfig.class);`
    - 이 클래스는 ApplicationContext 인터페이스의 구현체이다

### 스프링 컨테이너의 생성 과정

1. 스프링 컨테이너 생성

![Untitled](/assets/images/post/2023-08-07/1.png)

- `new AnnotationConfigApplicationContext(Appconfig.class)`
- 스프링 컨테이너를 생성할 때는 구성 정보를 지정해주어야한다.
    - 위에서는 `AppConfig.class` 를 구성 정보로 지정했다.

1. 스프링 빈 등록

![Untitled](/assets/images/post/2023-08-07/2.png)

- 스프링 컨테이너는 파라미터로 넘어온 설정 클래스 정보를 사용해서 스프링 빈을 등록한다.

**[ 빈 이름 ]** 

- 빈 이름은 메서드 이름을 사용한다.
- 빈 이름을 직접 부여할 수 도 있다.
    - `@Bean(name=”memberService2”)`
- **빈 이름은 항상 다른 이름을 부여해야한다.**
    - 다른 빈이 무시되거나 기존 빈을 덮어버리거나 설정에 따라 오류가 발생한다.

1. 스프링 빈 의존관계 설정 - 준비

![Untitled](/assets/images/post/2023-08-07/3.png)

1. 스프링 빈 의존관계 설정 - 완료

![Untitled](/assets/images/post/2023-08-07/4.png)

- 스프링 컨테이너는 설정 정보를 참고해서 의존관계를 주입(DI) 한다.

## 컨테이너에 등록된 모든 빈 조회

### 모든 빈 출력하기

- `AnnotationConfigApplicationContext ac;`
- `ac.getBeanDefinitionNames()` : 스프링에 등록된 모든 빈 이름을 조회한다.
- `ac.getBean()` : 빈 이름으로 빈 객체(인스턴스)를 조회한다.

### 애플리케이션 빈 출력하기

- 스프링이 내부에서 사용하는 빈은 제외하고, 직접 등록한 빈만 출력하는 방법
- 스프링이 내부에서 사용하는 빈은 `BeanDefinition`의 `getRole()`로 구분할 수 있다.
    - ROLE_APPLICATION - 스프링 내부에서 등록한 빈이 아니라 직접 등록한 빈
    - ROLE_INFRASTRUCTURE - 스프링 내부에서 등록한 빈

## 스프링 빈 조회 - 기본

- 스프링 컨테이너에서 스프링 빈을 찾는 가장 기본적인 조회 방법
    - ac.getBean(빈이름, 타입)
    - ac.getBean(타입)
- 조회 대상 스프링 빈이 없으면 예외 발생
    - `NoSuchBeanDefinitionException: No bean named 'xxxxx' available`

## 스프링 빈 조회 - 동일한 타입이 둘 이상

- 타입으로 조회시 같은 타입의 스프링 빈이 둘 이상이면 오류가 발생한다. 이때는 빈 이름을 지정하자.
- `ac.getBeansOfType()` 을 사용하면 해당 타입의 모든 빈을 조회할 수 있다.

## 스프링 빈 조회 - 상속관계

- 부모 타입으로 조회하면, 자식 타입도 함께 조회한다.
- 그래서 모든 자바 객체의 최고 부모인 `Object` 타입으로 조회하면, 모든 스프링 빈을 조회한다.

![Untitled](/assets/images/post/2023-08-07/5.png)

## BeanFactory와 ApplicationContext

![Untitled](/assets/images/post/2023-08-07/6.png)

### BeanFactory

- 스프링 컨테이너의 최상위 인터페이스다
- 스프링 빈을 관리하고 조회하는 역할을 담당한다.
- `getBean()`을 제공한다.

### ApplicationContext

- BeanFactory 기능을 모두 상속받아서 제공한다.
- 빈을 관리하고 검색하는 기능을 BeanFactory가 제공하는데, 둘의 차이는 뭘까
    - 애플리케이션을 개발할 때는 빈을 관리하고 조회하는 기능은 물론이고, 수 많은 부가기능이 필요하다.
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d695863d-59f2-4b6c-85f6-988370585e85/Untitled.png)
    
- **메시지소스를 활용한 국제화 기능**
    - 한국에서 접속하면 한국어로, 영어권에서 접속하면 영어로 출력
- **환경변수**
    - 로컬, 개발, 운영등을 구분해서 처리
- **애플리케이션 이벤트**
    - 이벤트를 발행하고 구독하는 모델을 편리하게 지원
- **편리한 리소스 조회**
    - 파일, 클래스패스, 외부 등에서 리소스를 편리하게 조회

### 정리

- ApplicationContext는 BeanFactory의 기능을 상속받는다.
- ApplicationContext는 빈 관리기능 + 편리한 부가 기능을 제공한다.
- BeanFactory를 직접 사용할 일은 거의 없다. 부가기능이 포함된 ApplicationContext를 사용한다.
- BeanFactory나 ApplicationContext를 스프링 컨테이너라 한다.

## 다양한 설정 형식 지원 - JAVA, XML

> 스프링 컨테이너는 다양한 형식의 설정 정보를 받아드릴 수 있게 유연하게 설계되어 있다.
> 

![Untitled](/assets/images/post/2023-08-07/7.png)

### XML 설정 사용

- 최근에는 스프링 부트를 많이 사용하면서 XML기반의 설정은 잘 사용하지 않는다. 아직 많은 레거시 프로젝트 들이 XML로 되어 있고, 또 XML을 사용하면 컴파일 없이 빈 설정 정보를 변경할 수 있는 장점도 있으므
로 한번쯤 배워두는 것도 괜찮다.
- GenericXmlApplicationContext 를 사용하면서 xml 설정 파일을 넘기면 된다.
- 필요한 경우 스프링 공식 레퍼런스 문서를 확인하자. [[ link ]](https://spring.io/projects/spring-framework)

## 스프링 빈 설정 메타 정보 - BeanDefinition

- `BeanDefinition` 이라는 추상화를 통해 스프링은 다양한 설정 형식을 지원한다!
- 쉽게 이야기해서 **역할과 구현을 개념적으로 나눈 것**이다!
    - XML을 읽어서 `BeanDefinition`을 만들면 된다.
    - 자바 코드를 읽어서 `BeanDefinition`을 만들면 된다.
    - 스프링 컨테이너는 자바 코드인지, XML인지 몰라도 된다. 오직 `BeanDefinition`만 알면 된다.
- `BeanDefinition` 을 빈 설정 메타정보라 한다.
    - @Bean , <bean> 당 각각 하나씩 메타 정보가 생성된다.
- 스프링 컨테이너는 이 메타정보를 기반으로 스프링 빈을 생성한다.

![Untitled](/assets/images/post/2023-08-07/8.png)

![Untitled](/assets/images/post/2023-08-07/9.png)

- 각각의 형식에 따라 AppConfig의 정보를 읽고 `BeanDefinition`을 생성한다.
- 새로운 형식의 설정 정보가 추가되는 경우 새로운 `BeanDefinitionReader`를 만들어서 BeanDefinition을 생성하면 된다.

### BeanDefinition

**[ BeanDefinition정보 ]**

- `BeanClassName`: 생성할 빈의 클래스 명(자바 설정 처럼 팩토리 역할의 빈을 사용하면 없음)
- `factoryBeanName`: 팩토리 역할의 빈을 사용할 경우 이름, 예) appConfig
- `factoryMethodName`: 빈을 생성할 팩토리 메서드 지정, 예) memberService
- `Scope`: 싱글톤(기본값)
- `lazyInit`: 스프링 컨테이너를 생성할 때 빈을 생성하는 것이 아니라, 실제 빈을 사용할 때 까지 최대한 생
성을 지연처리 하는지 여부
- `InitMethodName`: 빈을 생성하고, 의존관계를 적용한 뒤에 호출되는 초기화 메서드 명
- `DestroyMethodName`: 빈의 생명주기가 끝나서 제거하기 직전에 호출되는 메서드 명
- `Constructor arguments, Properties`: 의존관계 주입에서 사용한다. (자바 설정 처럼 팩토리 역할의 빈을 사용하면 없음)

## 코드 확인 [[ LINK ]](https://github.com/raindragonn/spring-basic-exam/pull/2) - 해당 pr의 커밋내역 확인