---
layout: "post"
title: "Spring 핵심 원리 - 스프링 빈 스코프"
subtitle: "Spring 핵심 원리 - 스프링 빈 스코프"
date: 2023-08-11
author: "raindragonn"
lang: ko
categories: [Spring]
tags:
  - Spring
  - Spring Boot
  - 기본
---

# 빈 스코프

## 빈 스코프란?

- 스프링 빈은 기본적으로 싱글톤 스코프로 생성된다.
- 스코프는 번역 그대로 빈이 존재할 수 있는 범위를 뜻한다.

### 빈 스코프의 종류

- **[ 싱글톤 ]**
    - 기본 스코프로, 스프링 컨테이너의 시작과 종료까지 유지되는 가장 넓은 범위의 스코프이다.
- **[ 프로토 타입 ]**
    - 스프링 컨테이너는 프로토타입 빈의 생성과 의존관계 주입까지만 관여하고 더는 관리하지 않는 매우 짧은 범위의 스코프
- **웹 관련 스코프**
    - **[ request ]**: 웹 요청이 들어오고 나갈때(response) 까지 유지되는 스코프
    - **session**: 웹 세션이 생성되고 종료될 때 까지 유지되느 스코프
    - **application**: 웹의 서블릿 컨텍스트와 같은 범위로 유지되는 스코프

### 빈 스코프의 지정 방법

1. 컴포넌트 스캔 자동 등록
    
    ```java
    @Scope("prototype")
    @Component
    public class PrototypeBean { }
    ```
    
2. 수동 등록 
    
    ```java
    @Scope("prototype")
    @Bean
    PrototypeBean prototypeBean() {
    		return new PrototypeBean();
    }
    ```
    

## 프로토타입 스코프

- 프로토타입 스코프를 스프링 컨테이너에 조회하면 스프링 컨테이너는 항상 새로운 인스턴스를 생성해서 반환한다.

![Untitled](/assets/images/post/2023-08-11/1.png)

![Untitled](/assets/images/post/2023-08-11/2.png)

- 스프링 컨테이너는 생성한 프로토타입 빈을 클라이언트에 반환한다.
- 이후에 스프링 컨테이너에 **같은 요청이 오면 항상 새로운 프로토타입 빈을 생성해서 반환**한다.

**[ 정리 ]**

- **스프링 컨테이너는 프로토타입 빈을 생성하고, 의존관계 주입, 초기화까지만 처리한다.**
    - 그에따라 `@PreDestroy` 와 같은 소멸 콜백 메서드는 호출되지 않는다.

```java
public class PrototypeTest {

    @Test
    void prototypeBeanFind() {
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(PrototypeBean.class);

        System.out.println("find prototypeBean1");
        PrototypeBean prototypeBean1 = ac.getBean(PrototypeBean.class);

        System.out.println("find prototypeBean2");
        PrototypeBean prototypeBean2 = ac.getBean(PrototypeBean.class);

        System.out.println("prototypeBean1 = " + prototypeBean1);
        System.out.println("prototypeBean2 = " + prototypeBean2);

				ac.close();
        Assertions.assertThat(prototypeBean1).isNotSameAs(prototypeBean2);
    }

    @Scope("prototype")
    static class PrototypeBean {

        @PostConstruct
        public void init() {
            System.out.println("PrototypeBean.init");
        }

        @PreDestroy
        public void destroy() {
            System.out.println("PrototypeBean.destroy");
        }

    }

}
------
find prototypeBean1
PrototypeBean.init
find prototypeBean2
PrototypeBean.init
prototypeBean1 = raindragonn.core.scope.PrototypeTest$PrototypeBean$$SpringCGLIB$$0@61eaec38
prototypeBean2 = raindragonn.core.scope.PrototypeTest$PrototypeBean$$SpringCGLIB$$0@125290e5
```

- 싱글톤 빈과는 다르게 프로토 타입 스코프의 빈은 스프링 컨테이너에서 빈을 조회할 때 생성되고, 초기화 메서드도 실행된다.
- 프로토타입 빈을 2번 조회했으므로 다른 인스턴스의 스프링 빈이 생성되고, 초기화도 2번 실행 된 것을 확인할 수 있다.
- 싱글톤 빈은 스프링 컨테이너가 관리하기 때문에 종료될 때 빈의 종료 메서드가 실행되지만, 프로토타입 빈은 스프링 컨테이너가 종료될 때 `@PreDestroy` 같은 소멸 콜백 메서드는 실행되지 않는다.
    - 프로토타입 빈의 소멸시 특정 동작이 필요한 경우 직접 호출을 해야한다.
    - ex) `prototypeBean1.destroy();`

### 프로토타입 빈의 특징

- 스프링 컨테이너에 요청할 때 마다 새로 생성된다.
- 스프링 컨테이너는 프로토타입 빈의 생성과 의존관계 주입 그리고 초기화까지만 관여한다.
- 종료 메서드가 호출되지 않는다.
- 프로토타입 빈을 조회한 클라이언트가 직접 해당 인스턴스를 관리해야 한다.
    - 종료 메서드에 대한 호출도 생명주기 콜백을 활용하지 못하기에 클라이언트가 직접 해야한다.
    

## 프로토타입 빈과 싱글톤 빈을 함께 사용시 문제점

- 싱글톤 빈과 함께 사용할 때는 의도한 대로 잘 동작하지 않으므로 주의해야 한다.

### 프로토타입 빈 직접 요청

![Untitled](/assets/images/post/2023-08-11/3.png)

1. 클라이언트A는 컨테이너에 프로토타입 빈을 요청한다.
2. 스프링 컨테이너는 빈을 새로 생성해서 반환하며 해당 빈의 count 필드의 값은 0이다.
3. 클라이언트는 조회한 프로토타입 빈에 `addCount()` 를 통해 count 필드의 값에 +1 한다.
4. 결과적으로 프로토타입 빈(**@x01**)의 count는 1이 된다.

### 프로토타입 빈 직접요청 2

![Untitled](/assets/images/post/2023-08-11/4.png)

1. 클라이언트B는 스프링 컨테이너에 프로토타입 빈을 요청한다.
2. 스프링 컨테이너는 프로토타입 빈을 새로 생성해서 반환(**x02**)한다. 해당 빈의 count 필드 값은 0이다.
3. 클라이언트는 조회한 프로토타입 빈에 `addCount()` 를 통해 count 필드의 값에 +1 한다.
4. 결과적으로 프로토타입 빈(**@x02**)의 count는 1이 된다.

### 싱글톤 빈에서 프로토타입 빈 사용 1

![Untitled](/assets/images/post/2023-08-11/5.png)

- `ClientBean` 은 싱글톤이므로, 스프링 컨테이너 생성 시점에 함께 생성되고, 의존관계 주입도 발생한다.
1. `clientBean` 은 의존관계 자동 주입을 사용한다. 주입 시점에 스프링 컨테이너에 프로토타입 빈을 요청
한다.
2. 스프링 컨테이너는 프로토타입 빈을 생성해서 `clientBean` 에 반환한다. 프로토타입 빈의 count 필드
값은 0이다.
3. 이제 `clientBean` 은 프로토타입 빈을 내부 필드에 보관한다. (정확히는 참조값을 보관한다.)

### 싱글톤 빈에서 프로토타입 빈 사용 2

![Untitled](/assets/images/post/2023-08-11/6.png)

1. 클라이언트 A는 `clientBean` 을 스프링 컨테이너에 요청해서 받는다.싱글톤이므로 항상 같은
`clientBean` 이 반환된다.
2. 클라이언트 A는 `clientBean.logic()` 을 호출한다.
3. `clientBean` 은 `prototypeBean`의 `addCount()` 를 호출해서 프로토타입 빈의 count를 증가한다.
count값이 1이 된다.

### 싱글톤 빈에서 프로토타입 빈 사용 3

![Untitled](/assets/images/post/2023-08-11/7.png)

1. 클라이언트 B는 `clientBean` 을 스프링 컨테이너에 요청해서 받는다.싱글톤이므로 항상 같은
`clientBean` 이 반환된다.
2. `clientBean`이 **내부에 가지고 있는 프로토타입 빈은 이미 과거에 주입이 끝난 빈이다.** 주입 시점에 스프링 컨테이너에 요청해서 프로토타입 빈이 새로 생성이 된 것이지, **사용 할 때마다 새로 생성되는 것이 아니다!**
3. 클라이언트 B는 clientBean.logic() 을 호출한다.
4. clientBean 은 prototypeBean의 addCount() 를 호출해서 프로토타입 빈의 count를 증가한다. 원
래 count 값이 1이었으므로 2가 된다.

### **[ 문제점 ]**

- 스프링은 일반적으로 싱글톤 빈을 사용하므로, 싱글톤 빈이 프로토타입 빈을 사용하게 된다.
- 싱글톤 빈은 **생성 시점에만 의존관계 주입을 받기에** **프로토타입 빈이 새로 생성되기는 하지만, 싱글톤 빈과 함께 계속 유지되는것이 문제다.**

## 프로토타입 빈과 싱글톤 빈을 함께 사용시 Provider로 문제 해결

### DL (Dependency Lookup)

- 의존관계를 외부에서 주입(DI)받는게 아니라 직접 필요한 의존관계를 찾는 것을 Dependency Lookup, 의존관계 조회(탐색) 이라한다.
    - ex) ApplicationContext를 주입받아 해당 ApplicationContext로 프로토타입 빈을 조회하는 것

### ObjectFactory, ObjectProvider

- 지정한 빈을 컨테이너에서 대신 찾아주는 DL 서비스를 제공하는 것이 바로 `ObjectProvider` 다.
- 과게어는 ObjectFactory가 있었는데, 여기에 편의 기능을 추가해서 `ObjectProvider` 가 만들어졌다.

```java
@Scope("singleton")
static class ClientBean {

    @Autowired
    private ObjectProvider<PrototypeBean> prototypeBeanProvider;

    public int logic() {
        PrototypeBean prototypeBean = prototypeBeanProvider.getObject();

        prototypeBean.addCount();
	      return prototypeBean.getCount();
    }
}
```

- `ObjectProvider` 의 `getObject()` 를 호출하면 내부에서는 스프링 컨테이너를 통해 해당 빈을 찾아서 반환한다. (**DL**)
- 스프링이 제공하는 기능을 사용하지만, 기능이 단순하므로 단위테스트를 만들거나 mock 코드를 만들기는
훨씬 쉬워진다.
- `ObjectProvider` 는 지금 딱 필요한 DL 정도의 기능만 제공한다.

**[ 특징 ]** 

- **스프링에 의존한다.**
- `ObjectFactory`: 기능이 단순, 별도의 라이브러리 필요 없음, 스프링에 의존
- `ObjectProvider`: `ObjectFactory` 상속, 옵션, 스트림 처리등 편의 기능이 많고, 별도의 라이브러리 필요 없음, 스프링에 의존

### JSR-330 Provider

- `javax.inject.Provider` 라는 JSR-330 자바 표준을 사용하는 방법이다.
- 라이브러리를 추가해야 사용가능하다.
    - 스프링부트 3.0 미만
        - `javax.inject:javax.inject:1` 라이브러리를 gradle에 추가해야 한다.
    - 스프링부트 3.0 이상
        - `jakarta.inject:jakarta.inject-api:2.0.1` 라이브러리를 gradle에 추가해야 한다.

```java
@Scope("singleton")
@RequiredArgsConstructor
static class ClientBean {

    private final Provider<PrototypeBean> prototypeBeanProvider;

    public int logic() {
        PrototypeBean prototypeBean = prototypeBeanProvider.get();

        prototypeBean.addCount();
        return prototypeBean.getCount();
    }
}
```

- 실행해보면 `provider.get()` 을 통해서 항상 새로운 프로토타입 빈이 생성되는 것을 확인할 수 있다.
- `provider` 의 `get()` 을 호출하면 내부에서는 스프링 컨테이너를 통해 해당 빈을 찾아서 반환한다. (**DL**)
- 자바 표준이고, 기능이 단순하므로 단위테스트를 만들거나 mock 코드를 만들기는 훨씬 쉬워진다.
- `Provider` 는 지금 딱 필요한 DL 정도의 기능만 제공한다.

**[ 특징 ]**

- get() 메서드 하나로 기능이 매우 단순하다.
- 별도의 라이브러리가 필요하다.
- 자바 표준이므로 스프링이 아닌 다른 컨테이너에서도 사용할 수 있다.

### 정리

- 프로토타입 빈을 언제 사용하는가?
    - 매번 사용할 때 마다 의존관계 주입이 완료된 새로운 객체가 필요한 경우에 사용하면 된다.
- `ObjectProvider` , `JSR-330 Provider` 등은 프로토타입 뿐만 아니라 DL이 필요한 경우는 언제든지 사용 할 수 있다.

## 웹 스코프

### 특징

- 웹 환경에서만 동작한다.
- 프로토타입과 다르게 스프링이 해당 스코프의 종료시점까지 관리한다.
    - 종료 메서드가 호출된다.

### 종류

1. **request** : HTTP 요청 하나가 들어오고 나갈 때 까지 유지되는 스코프, 각각의 HTTP 요청마다 별도의 인스턴스가 생성되고, 관리된다.
2. **session :** HTTP Session과 동일한 생명주기를 가지는 스코프
3. **application :** 서블릿 컨텍스트(ServletContext)와 동일한 생명주기를 가지는 스코프
4. **websocket:** 웹 소켓과 동일한 생명주기를 가지느 ㄴ스코프

### **HTTP request 요청 당 각각 할당되는 request 스코프**

![Untitled](/assets/images/post/2023-08-11/8.png)

- 동시에 요청을 하더라도 각 요청에 따른 각기 다른 인스턴스를(빈) 사용한다.

## request 스코프 예제 만들기

### **웹 환경 추가**

- 웹 스코프는 웹 환경에서만 동작하므로 web 환경이 동작하도록 라이브러리를 추가해야한다.

```Groovy
dependencies {
...
    implementation 'org.springframework.boot:spring-boot-starter-web'
...
}
```

- spring-boot-starter-web 라이브러리를 추가하면 스프링 부트는 내장 톰켓 서버를 활용해서 웹 서버와 스프링을 함께 실행시킨다.
- 스프링 부트는 웹 라이브러리가 없으면 `AnnotationConfigApplicationContext` 을 기반으로 애플리케이션을 구동한다.
- 웹 라이브러리가 추가 되면 웹과 관련된 추가 설정과 환경들이 필요하므로`AnnotationConfigServletWebServerApplicationContext` 를 기반으로 애플리케이션을 구동한다.
- 만약 기본 포트인 8080 포트를 다른곳에서 사용중이어서 오류가 발생하면 포트를 변경해야 한다.
    
    ```
    server.port=9090
    ```
    

### request 스코프 예제 개발

- 동시에 여러 HTTP 요청이 온 경우 정확히 어떤 요청이 남긴 로그인지 구분할 때 request 스코프를 활용하면 좋다.
- 기대하는 공통 포맷
    - [UUID][requestURL] {message}
    - UUID를 사용해서 HTTP 요청을 구분하자.
    - requestURL로 어떤 URL을 요청해서 남은 로그인지 확인하자.

```java
@Component
@Scope(value = "request")
public class MyLogger {

    private String uuid;
    private String requestURL;

    public void setRequestURL(String requestURL) {
        this.requestURL = requestURL;
    }

    public void log(String message) {
        System.out.println("[" + uuid + "]" + "[" + requestURL + "] " + message);
    }

    @PostConstruct
    public void init() {
        // 랜덤 아이디 생성
        String uuid = UUID.randomUUID().toString();
        System.out.println("[" + uuid + "] request scope bean create: " + this);
    }

    @PreDestroy
    public void close() {
        System.out.println();
        System.out.println("[" + uuid + "] request scope bean close: " + this);
    }
}
```

- 로그를 출력하기 위한 MyLogger 클래스이다.
- `@Scope(value = "request")` 를 사용해서 request 스코프로 지정했다. 이제 이 빈은 HTTP 요청 당 하나씩 생성되고, HTTP 요청이 끝나는 시점에 소멸된다.
- 이 빈이 생성되는 시점에 자동으로 `@PostConstruct` 초기화 메서드를 사용해서 `uuid`를 생성해서 저장해
둔다. 이 빈은 HTTP 요청 당 하나씩 생성되므로, uuid를 저장해두면 다른 HTTP 요청과 구분할 수 있다.
- 이 빈이 소멸되는 시점에 `@PreDestroy` 를 사용해서 종료 메시지를 남긴다.
- `requestURL` 은 이 빈이 생성되는 시점에는 알 수 없으므로, 외부에서 `setter`로 입력 받는다.

### 데모 컨트롤러 생성

```java
@Controller
@RequiredArgsConstructor
public class LogDemoController {

    private final LogDemoService logDemoService;
    private final MyLogger myLogger;

    @RequestMapping("log-demo")
    @ResponseBody
    public String logDemo(HttpServletRequest request) {
        String requestURL = request.getRequestURL().toString();
        myLogger.setRequestURL(requestURL);
        myLogger.log("controller test");
        logDemoService.logic("testID");
        return "OK";
    }
}
```

- 로거가 잘 작동하는지 확인하는 테스트용 컨트롤러다.
- 여기서 `HttpServletRequest`를 통해서 요청 URL을 받았다.
    - requestURL 값 http://localhost:8080/log-demo
- 이렇게 받은 `requestURL` 값을 `myLogger`에 저장해둔다. myLogger는 HTTP 요청 당 각각 구분되므로
다른 HTTP 요청 때문에 값이 섞이는 걱정은 하지 않아도 된다.
- 컨트롤러에서 controller test라는 로그를 남긴다.

### 데모 서비스 생성

```java
@Service
@RequiredArgsConstructor
public class LogDemoService {

    private final MyLogger myLogger;

    public void logic(String id) {
        myLogger.log("service id = " + id);
    }
}
```

- 비즈니스 로직이 있는 서비스 계층에서도 로그를 출력해보자.
- 여기서 중요한점이 있다. request scope를 사용하지 않고 파라미터로 이 모든 정보를 서비스 계층에 넘긴
다면, 파라미터가 많아서 지저분해진다. 더 문제는 `requestURL` 같은 웹과 관련된 정보가 웹과 관련없는 서
비스 계층까지 넘어가게 된다. 웹과 관련된 부분은 컨트롤러까지만 사용해야 한다. **서비스 계층은 웹 기술에
종속되지 않고, 가급적 순수하게 유지하는 것이 유지보수 관점에서 좋다.**
- request scope의 `MyLogger` 덕분에 이런 부분을 파라미터로 넘기지 않고, `MyLogger`의 멤버변수에 저
장해서 코드와 계층을 깔끔하게 유지할 수 있다.

### **실제는 기대와 다르게 애플리케이션 실행 시점에 오류 발생**

`Error creating bean with name 'myLogger': Scope 'request' is not active for the
  current thread; consider defining a scoped proxy for this bean if you intend to
  refer to it from a singleton;`

- 스프링 애플리케이션을 실행 시키면 오류가 발생한다.
- request 스코프 빈은 아직 생성되지 않는다.
    - 실제 요청이 와야 생성할 수 있지만 현재 생성자 주입으로 받으려했다.

## 스코프와 Provider

```java
@Controller
@RequiredArgsConstructor
public class LogDemoController {

    private final LogDemoService logDemoService;
    private final ObjectProvider<MyLogger> myLoggerProvider;

    @RequestMapping("log-demo")
    @ResponseBody
    public String logDemo(HttpServletRequest request) {
        MyLogger myLogger = myLoggerProvider.getObject();
        String requestURL = request.getRequestURL().toString();

        myLogger.setRequestURL(requestURL);
        myLogger.log("controller test");

        logDemoService.logic("testID");
        return "OK";
    }
}
```

```java
@Service
@RequiredArgsConstructor
public class LogDemoService {

    private final ObjectProvider<MyLogger> myLoggerProvider;

    public void logic(String id) {
        MyLogger myLogger = myLoggerProvider.getObject();
        myLogger.log("service id = " + id);
    }
}
```

- `ObjectProvider` 덕분에 `ObjectProvider.getObject()` 를 호출하는 시점까지 request scope **빈의
생성을 지연**할 수 있다.
    - **정확히는 요청을 지연한다.**
- `ObjectProvider.getObject()` 를 호출하시는 시점에는 HTTP 요청이 진행중이므로 request scope
빈의 생성이 정상 처리된다.
- `ObjectProvider.getObject()` 를 `LogDemoController` , `LogDemoService` 에서 각각 한번씩 따로 호출해도 **같은 HTTP 요청이면 같은 스프링 빈이 반환**된다.

## 스코프와 프록시

```java
@Component
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class MyLogger {
...
}
```

- `proxyMode = ScopedProxyMode.TARGET_CLASS` 를 추가해주자.
    - 여기가 핵심이다.
- 적용 대상이 인터페이스가 아닌 클래스면 `ScopedProxyMode.TARGET_CLASS` 를 선택
- 적용 대상이 인터페이스면 `ScopedProxyMode.INTERFACES` 를 선택
- 이렇게 하면 `MyLogger`의 가짜 프록시 클래스를 만들어두고 HTTP request와 상관 없이 가짜 프록시 클
래스를 다른 빈에 미리 주입해 둘 수 있다.
- **Controller와 Service에서는 Provider사용 이전으로 돌려준다.**

### 웹 스코프와 프록시 동작 원리

- `**CGLIB` 라이브러리를 통해 내 클래스를 상속 받은 가짜 프록시 객체를 만들어서 주입한다.**
- `@Scope` 의 `proxyMode = ScopedProxyMode.TARGET_CLASS` 를 설정하면 스프링 컨테이너는 `CGLIB` 라는 바이트코드를 조작하는 라이브러리를 사용해서, MyLogger를 상속받은 가짜 프록시 객체를 생성한다.
- 결과를 확인해보면 우리가 등록한 순수한 `MyLogger` 클래스가 아니라`MyLogger$$EnhancerBySpringCGLIB` 이라는 클래스로 만들어진 객체가 대신 등록된 것을 확인할 수 있다.
- 그리고 스프링 컨테이너에 "myLogger"라는 이름으로 진짜 대신에 이 가짜 프록시 객체를 등록한다.
- ac.getBean("myLogger", MyLogger.class) 로 조회해도 프록시 객체가 조회되는 것을 확인할 수 있다.
- 그래서 의존관계 주입도 이 가짜 프록시 객체가 주입된다.

![Untitled](/assets/images/post/2023-08-11/9.png)

**[ 가짜 프록시 객체는 요청이 오면 그때 내부에서 진짜 빈을 요청하는 위임 로직이 들어있다. ]**

- 가짜 프록시 객체는 내부에 진짜 myLogger를 찾는 방법을 알고 있다.
- 클라이언트가 myLogger.logic() 을 호출하면 사실은 가짜 프록시 객체의 메서드를 호출한 것이다.
- 가짜 프록시 객체는 request 스코프의 진짜 myLogger.logic() 를 호출한다.
- 가짜 프록시 객체는 원본 클래스를 상속 받아서 만들어졌기 때문에 이 객체를 사용하는 클라이언트 입장에 서는 사실 원본인지 아닌지도 모르게, 동일하게 사용할 수 있다(다형성)

**[ 동작 정리 ]**

1. `CGLIB` 라이브러리를 통해 내 클래스를 상속 받은 가짜 프록시 객체를 만들어서 주입한다.
2. 해당 프록시 객체는 실제 요청이 오면 그때 내부에서 실제 빈을 요청하는 위임 로직이 있다.
3. 가짜 프록시 객체는 실제 request scope와 관계 없다.
    - 내부에 단순한 위임 로직만 있고, 싱글톤 처럼 동작한다.

**[ 특징 정리 ]**

- 프록시 객체 덕분에 클라이언트는 마치 싱글톤 빈을 사용하듯이 편리하게 request scope를 사용할 수 있
다.
- **사실 `Provider`를 사용하든, 프록시를 사용하든 핵심은 진짜 객체 조회를 꼭 필요한 시점까지 지연
처리 한다는 점이다.**
- 단지 어노테이션 설정 변경만으로 원본 객체를 프록시 객체로 대체할 수 있다. 이것이 바로 다형성과 DI 컨
테이너가 가진 큰 강점이다.
- 꼭 웹 스코프가 아니어도 프록시는 사용할 수 있다.

****************************[ 주의점 ]****************************

- 마치 싱글톤을 사용하는 것 같지만 다르게 동작하기 때문에 주의해서 사용해야 한다.
- request와 같은 특별한 스코프는 꼭 필요한 곳에서만 최소화해서 사용하자, 무분별하게 사용하면 유지보수하기 어려워진다.

## 코드확인 [[ LINK ]](https://github.com/raindragonn/spring-basic-exam/pull/7) - 해당 PR의 커밋 확인하기.