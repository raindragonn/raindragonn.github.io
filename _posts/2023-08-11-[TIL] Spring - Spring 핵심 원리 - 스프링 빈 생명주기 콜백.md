---
layout: "post"
title: "Spring 핵심 원리 - 스프링 빈 생명주기 콜백"
subtitle: "Spring 핵심 원리 - 스프링 빈 생명주기 콜백"
date: 2023-08-11
author: "raindragonn"
lang: ko
categories: [Spring]
tags:
  - Spring
  - Spring Boot
  - 기본
---

## 빈 생명주기 콜백 시작

- 데이터베이스 커넥션 풀이나, 네트워크 소켓처럼 애플리케이션 시작 시점에 필요한 연결을 미리하거나 애플리케이션 종료 시점에 연결을 모두 종료하는 작업을 진행하는 경우 객체의 초기화 혹은 종료 작업이 필요하다.

### 네트우크 연결을 가정한 예제

- 애플리케이션 시작 시점에 `connect()` 를 통해 연결을 맺어야하고, 애플리케이션 종료시 `disconnect()` 를 통해 연결을 끊어야 한다.

```java
public class NetworkClient {

    private String url;

    public NetworkClient() {
        System.out.println("생성자 호출, url = " +url);
        connect();
        call("초기화 연결 메시지");
    }

    public void setUrl(String url) {
        this.url = url;
    }

    //서비스 시작 시 호출
    public void connect() {
        System.out.println("connect: " + url);
    }

    public void call(String message) {
        System.out.println("call: " + url + " message = " + message);
    }

    //서비스 종료 시 호출
    public void disConnect() {
        System.out.println("close: " + url);
    }
}
```

```java
public class BeanLifeCycleTest {

    @Test
    void lifeCycleTest() {
        ConfigurableApplicationContext ac = new AnnotationConfigApplicationContext(LifeCycleConfig.class);
        NetworkClient client = ac.getBean(NetworkClient.class);
        ac.close();
    }

    @Configuration
    static class LifeCycleConfig {

        @Bean
        public NetworkClient networkClient() {
            NetworkClient networkClient = new NetworkClient();
            networkClient.setUrl("http://hello-spring.dev");
            return networkClient;
        }
    }
}

-------------
생성자 호출, url = null
connect: null
call: null message = 초기화 연결 메시지
```

- `NetworkClient` 의 생성자를 보면 url 정보 없이 `connect` 가 호출되는 것을 확인할 수 있다.

### 스프링 빈의 라이프사이클

- **객체 생성 → 의존관계 주입**
    - 생성자 주입의 경우를 제외한 setter, 필드 주입 등의 경우
- 스프링 빈은 객체를 생성하고, 의존관계 주입이 다 끝난 다음에야 필요한 데이터를 사용할 수 있는 준비가 완료된다.
    - 초기화 작업은 의존관계 주입이 모두 완료되고 난 다음에 호출해야 한다.
- **스프링은 의존관계 주입이 완료되면 스프링 빈에게 콜백 메서드를 통해서 초기화 시점을 알려주는 다양한 기능을 제공한다.**
    - 또한 스프링은 **스프링 컨테이너가 종료되기 직전에 소멸 콜백**을 준다.

### 스프링 빈의 이벤트 라이프 사이클

- 라이프 사이클의 순서는 아래와 같다.
1. 스프링 컨테이너 생성
2. 스프링 빈 생성
3. 의존관계 주입
4. 초기화 콜백
    - 빈이 생성되고, 빈의 의존관계 주입이 완료된 후 호출
5. 사용
6. 소멸 전 콜백
    - 빈이 소멸되기 직전에 호출
7. 스프링 종료

### 객체의 생성과 초기화를 분리하자.

- **생성자**는 필수 정보(파라미터)를 받고, 메모리를 할당해서 객체를 생성하는 책임을 가진다.
    - 생성자는 객체를 생성하는것에 집중해야한다.
    - 다만, 내부 값들만 약간 변경하는 정도로 단순한 경우에는 생성자에서 처리하는게 더 나을 수도 있다.
- **초기화**는 생성된 값들을 활용해서 외부 커넥션을 연결하는 등 무거운 동작을 수행한다.
    - 초기화는 일련의 동작으로서, 생성자와는 다른 책임을 가진다.

### 스프링은 크게 3가지의 방법으로 빈 생명주기 콜백을 지원한다.

1. 인터페이스 (`InitializingBean`, `DisposableBean`)
2. 설정 정보에 초기화 메서드, 종료 메서드 지정
3. `@PostConstruct`, `@PreDestroy` 어노테이션 활용

## 인터페이스 - InitializingBean, DisposableBean

```java
public class NetworkClient implements InitializingBean, DisposableBean {

    private String url;

    public NetworkClient() {
        System.out.println("생성자 호출, url = " + url);
    }

    public void setUrl(String url) {
        this.url = url;
    }

    //서비스 시작 시 호출
    public void connect() {
        System.out.println("connect: " + url);
    }

    public void call(String message) {
        System.out.println("call: " + url + " message = " + message);
    }

    //서비스 종료 시 호출
    public void disConnect() {
        System.out.println("close: " + url);
    }

    // 의존관계 주입이 끝날때 콜백
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("NetworkClient.afterPropertiesSet");
        connect();
        call("초기화 연결 메시지");
    }

    // 빈 생명주기가 종료될 때 콜백
    @Override
    public void destroy() throws Exception {
        System.out.println("NetworkClient.destroy");
        disConnect();
    }
}
```

- `InitializingBean` 은 `afterPropertiesSet()` 메서드를 통해 의존관계 주입이 끝날때 즉, 초기화 시점에 콜백을 지원한다.
- `DisposableBean` 은 `destroy()` 메서드를 통해 빈 생명주기가 종료될 때 즉, 소멸 시점에 콜백을 지원한다.

### 초기화, 소멸 인터페이스의 단점

- 해당 인터페이스는 스프링 전용 인터페이스로 해당 코드가 스프링 전용 인터페이스에 의존한다.
- 초기화, 소멸 메서드의 이름을 변경할 수 없다.
- 내가 코드를 고칠 수 없는 외부 라이브러리에 적용할 수 없다.
- **최근에는 거의 사용하지 않는다.**

## 빈 등록 초기화, 소멸 메서드 지정

- 수동 등록 시 설정 정보에 `@Bean(initMethod = “init”, destroyMethod = “close”)` 처럼 초기화 및 소멸 메서드를 지정할 수 있다.

```java
public class NetworkClient {

		...

    // 의존관계 주입이 끝날때 동작
    public void init() {
        System.out.println("NetworkClient.init");
        connect();
        call("초기화 연결 메시지");
    }

    // 빈 생명주기가 종료될 때 동작
    public void close() {
        System.out.println("NetworkClient.close");
        disConnect();
    }
}
```

```java
public class BeanLifeCycleTest {

    @Test
    void lifeCycleTest() {
        ConfigurableApplicationContext ac = new AnnotationConfigApplicationContext(LifeCycleConfig.class);
        NetworkClient client = ac.getBean(NetworkClient.class);
        ac.close();
    }

    @Configuration
    static class LifeCycleConfig {

        @Bean(initMethod = "init", destroyMethod = "close")
        public NetworkClient networkClient() {
            NetworkClient networkClient = new NetworkClient();
            networkClient.setUrl("http://hello-spring.dev");
            return networkClient;
        }
    }
}
```

- 메서드 이름을 자유롭게 사용할 수 있다.
- 스프링 빈이 스프링 코드에 의존하지 않는다.
- 코드가 아니라 설정 정보를 사용하기 때문에 코드를 고칠 수 없는 외부 라이브러리에도 초기화, 종료 메서드를 적용할 수 있다.

### 종료 메서드 추론 - destroyMethod 옵션의 특징

- `@Bean` 의 `destroyMethod` 속성에 특별한 기능이 있다.
    - 기본 값이 추론을 의미하는 “(inferred)” 로 등록되어 있다.
    - 라이브러리 대부분은 “close”, “shutdown” 이라는 이름의 종료 메서드를 사용한다.
    - 추론 기능을 통해 “close”, “shutdown” 라는 이름의 메서드를 자동으로 호출해준다.
    - 해당 추론기능을 사용하지 않는 경우 `destroyMethod = “”` 처럼 공백을 지정하면 된다.

## @PostConstruct, @PreDestroy 어노테이션

```java
public class NetworkClient {

		...

    // 의존관계 주입이 끝날때 동작
    @PostConstruct
    public void init() {
        System.out.println("NetworkClient.init");
        connect();
        call("초기화 연결 메시지");
    }

    // 빈 생명주기가 종료될 때 동작
    @PreDestroy
    public void close() {
        System.out.println("NetworkClient.close");
        disConnect();
    }
}
```

- `@PostConstruct`**,** `@PreDestroy` 어노테이션을 사용하면 간편하게 초기화 및 종료 콜백을 활용할 수 있다.

### [ 특징 ]

- **최신 스프링에서 권장하는 방법이다. - 해당 방식을 사용하자**
- 어노테이션 하나만 붙이면 되므로 매우 편리하다.
- 패키지가 `javax.annotation` 이다.
    - 스프링에 종속적인 기술이 아니라 자바 표준이다.
- **컴포넌트 스캔과 잘 어울린다.**
    - 수동 빈 주입을 사용하지 않아도 활용가능하다.
- 외부 라이브러리에 적용하지 못하는 단점이 존재한다.
    - **외부 라이브러리에서 초기화 및 종료가 필요한 경우에는 `@Bean` 옵션을 사용하자!**
    

## 코드 확인 [[ LINK ]](https://github.com/raindragonn/spring-basic-exam/pull/6) - 해당 PR의 커밋내역 확인.