---
layout: "post"
title: "Spring 입문 4. 스프링빈과 의존관계"
subtitle: "Spring 입문 4. 스프링빈과 의존관계"
date: 2023-07-14
author: "raindragonn"
lang: ko
categories: [Spring]
tags:
  - Spring
  - Spring Boot
  - 입문
  - DI
---
## 스프링빈과 의존관계

참고 강의:https://www.inflearn.com/roadmaps/373 

## 스프링 빈이란?

- 스프링 컨테이너에 의해 관리되는 재사용 가능한 소프트웨어 컴포넌트를 의미한다.
    - 인스턴스화된 객체를 의미
    - 기본으로 싱글톤으로 등록한다. (같은 스프링 빈인 경우 같은 인스턴스를 의미)
        - 설정으로 싱글톤이 아니게 설정할 수 있지만, 특별한 경우를 제외하면 싱글톤을 사용함.

## 스프링 빈을 등록하는 2가지 방법

### 1. 컴포넌트 스캔과 자동 의존관계 설정

- `@Autowired`, `@Component` 어노테이션을 이용하는 방법
- `@Component` 어노테이션이 붙은 경우 스프링 컨테이너에서 인스턴스를 관리한다. (스프링 빈으로 자동 등록된다.)
    - `@Controller`, `@Service`, `@Repository` 어노테이션은 `@Component` 어노테이션을 포함한다.
- 컴포넌트 스캔의 범위는 애플리케이션(`@SpringBootApplication`)을 포함한 패키지의 하위만이다.
    - `@SpringBootApplication` 가 포함하는 `@ComponentScan` 참고

### 2. 자바 코드로 직접 스프링 빈 등록하는 방법

- 직접적으로 스프링에 등록함.
- 상황에 따라 구현클래스를 변경해야 하는 경우 주로 사용된다.
    - 데이터 저장소의 변경 등
- `@Controller`는 예외
- `@Configuration` 어노테이션을 포함한 설정 클래스를 통해 등록하게된다.
- `@Bean` 어노테이션을 통해 컴포넌트를 추가한다.
    
    ```kotlin
    @Bean
    public MemberService memberService() {
    		return new MemberService(memberRepository());
    }
    ```
    

## 스프링 빈을 등록하고, 의존관계 설정하기

- 일반적으로 Controller는 Service를 Service는 Repository를 의존한다.
- 스프링이 실행될 때 스프링 컨테이너에서 @Controller 어노테이션이 붙은 컨트롤러의 객체를 생성 및 관리함.
    - Controller에서 사용하는 Service를 개별적으로 생성해서 사용하는것은 옳지않다.(하나의 객체를 공용으로 사용하도록)
    - 스프링 컨테이너에 Service를 등록해서 사용하는것이 옳다.
- Controller의 생성자에 `@Autowired` 어노테이션을 사용하는 경우 스프링 컨테이너에서 알맞은 서비스를 제공한다.(생성자뿐이 아니라 필드 주입, 세터를 통해서도 가능하지만 생성자를 이용하는게 의존관계가 실행중에 동적으로 변경하는 경우가 없기에 가장 안전하다.)
    - 위와 같은 방식을 사용하기 위해서는 컨테이너로 부터 제공받기위한 Service는 `@Service` 어노테이션이 필요하다.(`@Component`)
    - Service에서 사용하는 Repository 는 `@Repository` 어노테이션이 필요하다.(`@Component`)
    - 이처럼 필요로 하는 의존관계를 외부에서 넣어주는것을 **DI(Dependency Injection)**라고 한다.
- `@Autowired` 어노테이션은 스프링이 관리하지 않는(스프링에 등록되지 않거나, 스프링 컨테이너에 올라가지 않은)경우 동작하지 않는다.

### 스프링 빈 등록 이미지

![Untitled](/assets/images/post/2023-07-14-1.png)