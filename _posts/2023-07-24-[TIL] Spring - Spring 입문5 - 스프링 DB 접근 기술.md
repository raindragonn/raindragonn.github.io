---
layout: "post"
title: "Spring 입문5. 스프링 DB 접근 기술"
subtitle: "Spring 입문5. 스프링 DB 접근 기술"
date: 2023-07-24
author: "raindragonn"
lang: ko
categories: [Spring]
tags:
  - Spring
  - Spring Boot
  - 입문
---

참고 강의:https://www.inflearn.com/roadmaps/373 

# 스프링 DB 접근 기술

## H2 데이터베이스 설치

- [https://www.h2database.com](https://www.h2database.com/)
- 다운로드 및 설치
- h2 데이터베이스 버전은 스프링 부트 버전에 맞춘다.
- 권한 주기: chmod 755 [h2.sh](http://h2.sh/) (윈도우 사용자는 x)
- 실행: ./h2.sh (윈도우 사용자는 h2.bat)
- 데이터베이스 파일 생성 방법
    - jdbc:h2:~/test (최초 한번)
    - ~/test.mv.db 파일 생성 확인
    - 이후부터는 jdbc:h2:tcp://localhost/~/test 이렇게 접속

## 순수 JDBC

- 데이터 저장기술의 과거 방법 중 하나
- Application과 DB를 연결, DB에 쿼리를 통해 데이터를 관리
- build.gradle에 jdbc, h2 데이터베이스 관련 라이브러리 추가
    
    ```groovy
    implementation 'org.springframework.boot:spring-boot-starter-jdbc'
    runtimeOnly 'com.h2database:h2'
    ```
    
- 스프링 부트 데이터베이스 연결 설정 추가
    - resources/application.properties
    
    ```
    spring.datasource.url=jdbc:h2:tcp://localhost/~/test
    spring.datasource.driver-class-name=org.h2.Driver
    spring.datasource.username=sa
    ```
    

## 스프링 통합 테스트

- 스프링 컨테이너와 DB까지 연결한 통합 테스트 진행 방법

### 트랜잭션

- 데이터베이스의 상태를 변환시키는 하나의 논리적 기능을 수행하기 위한 작업의 단위 또는 한꺼번에 모두 수행되어야 할 일련의 연산들을 의미

### 스프링 통합테스트 방법

`@SpringBootTest`

- 스프링 컨테이너와 테스트를 함께 실행한다.

`@Transactional`

- 테스트 케이스에 이 어노테이션을 사용하면, 테스트 시작 전에 트랜잭션을 시작하고, 테스트 완료 후에 항상 롤백한다. 이렇게 하면 **DB에 데이터가 남지 않으므로 다음 테스트에 영향을 주지않는다.**
    - 테스트 케이스에서만 동작
    - `@commit` 어노테이션을 사용하면 실제로 해당 트랜잭션을 커밋한다.

## 스프링 JdbcTemplate

- 순수 JDBC와 ㅇ동일한 환경설정을 하면 된다.
- 스프링 JdbcTemplate과 MyBatis 같은 라이브러리는 JDBC API에서 본 반복 코드를 대부분 제거한다.
    - SQL은 직접 작성해야한다.

## JPA

- JPA는 기존의 반복 코드와 기본적인 SQL을 직접 만들어서 실행해준다.
- JPA를 사용하면 SQL과 데이터 중심의 설계에서 객체 중심의 설계로 패러다임을 전환할 수 있다.
- JPA를 사용하면 개발 생산성을 크게 높일 수 있다.

### 라이브러리 추가

```groovy
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
```

- spring-boot-starter-data-jpa 는 내부에 jdbc 관련 라이브러리를 포함한다. 따라서 jdbc는 제거해도된다.

### 스프링 부트에 JPA 설정 추가

```
spring.jpa.show-sql=true
spring.jpa.hibernate.ddl-auto=none
```

- show-sql : JPA가 생성하는 SQL을 출력한다.
- ddl-auto : JPA는 테이블을 자동으로 생성하는 기능을 제공하는데 none 를 사용하면 해당 기능을 끈다.
create 를 사용하면 엔티티 정보를 바탕으로 테이블도 직접 생성해준다. 해보자.

### 간단한 동작 원리

- 어노테이션을 통해 테이블로 사용하고자 하는 클래스를 지정,각 컬럼 별 필요 옵션 등을 설정할 수 있다.
    - ex) `@Entity`, `@Id`, `@GeneratedValue`, `@Column`

### 서비스 계층에 트랜잭션 추가

- `@Transactional`
- 스프링은 해당 클래스의 메서드를 실행할 때 트랜잭션을 시작하고, 메서드가 정상 종료되면 트랜잭션을커밋한다.
    - 만약 런타임 예외가 발생하면 롤백한다.
- **JPA를 통한 모든 데이터 변경은 트랜잭션 안에서 실행해야 한다.**

## 스프링 데이터 JPA

- 리포지토리에 구현 클래스 없이 인터페이스 만으로도 개발을 완료할 수 있다.
- 반복 개발해온 기본CRUD 기능도 스프링 데이터 JPA가 모두 제공한다.
- 스프링 데이터 JPA는 JPA를 편리하게 사용하도록 도와주는 기술이다.
- `JpaRepository` 를 상속한 interface를 만드는 경우 스프링 데이터 JPA가 SpringDataJpaMemberRepository 를 스프링 빈으로 자동 등록해준다.
- 인터페이스를 통한 기본적인 CRUD, findByName() , findByEmail() 처럼 메서드 이름 만으로 조회 기능, 페이징 기능 자동 제공

### 팁

- 사용한 sql 문들은 프로젝트에 .sql 파일에 정리를 해두는 편이 추후 편하다.