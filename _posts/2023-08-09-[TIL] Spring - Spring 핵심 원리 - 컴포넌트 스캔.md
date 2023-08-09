---
layout: "post"
title: "Spring 핵심 원리 - 컴포넌트 스캔"
subtitle: "Spring 핵심 원리 - 컴포넌트 스캔"
date: 2023-08-09
author: "raindragonn"
lang: ko
categories: [Spring]
tags:
  - Spring
  - Spring Boot
  - 기본
---

## 컴포넌트 스캔과 의존관계 자동 주입 시작하기

- 등록해야 할 스프링 빈이 수십, 수백개가 되면 일일이 등록하기도 귀 찮고, 설정 정보도 커지고, 누락하는 문제도 발생한다.
- 그래서 스프링은 설정 정보가 없어도 자동으로 스프링 빈을 등록하는 컴포넌트 스캔이라는 기능을 제공한
다.
- 의존관계도 자동으로 주입하는 `@Autowired` 라는 기능도 제공한다.

### 컴포넌트 스캔을 사용한 Configuration

```java
// ComponentScan 을 사용하면 @Component 어노테이션이 붙은 클래스를 자동으로 찾아준다.
// 옵션을 통해(파라미터) 스캔에 포함하고자 하는 클래스, 빼고자 하는 클래스를 지정이 가능하다.
@Configuration
@ComponentScan(excludeFilters = @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = Configuration.class))
public class AutoAppConfig {

}
```

- 컴포넌트 스캔을 사용하려면 `@ComponentScan` 을 설정 정보에 붙여주면 된다.
- 컴포넌트 스캔은 이름 그대로 `@Component` 어노테이션이 붙은 클래스를 스캔해서 스프링 빈으로 등록한다.
- `@Configuration` 이 컴포넌트 스캔의 대상이 된 이유도 `@Configuration` 소스코드를 열어보면`@Component` 어노테이션이 붙어있기 때문이다.

### 각 스프링 빈으로 등록하기 위한 @Component 추가

```java
@Component
public class MemoryMemberRepository implements MemberRepository {}

@Component
public class RateDiscountPolicy implements DiscountPolicy {}
```

### 의존관계 주입을 위한 생성자에 @Autowired 추가

```java
@Component
public class MemberServiceImpl implements MemberService {

    private final MemberRepository memberRepository;

    @Autowired
    public MemberServiceImpl(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
	...
}

@Component
public class OrderServiceImpl implements OrderService {

    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;

		@Autowired
    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
		...
}
```

- 이전에 AppConfig에서는 `@Bean` 으로 직접 설정 정보를 작성했고, 의존관계도 직접 명시했다. 이제는 이런 설정 정보 자체가 없기 때문에  `@Autowired` 를 통해 자동으로 주입해준다.
- `@Autowired` 를 사용하면 생성자에서 여러 의존관계도 한번에 주입받을 수 있다.

### 1. @ComponentScan

![Untitled](/assets/images/post/2023-08-09/1.png)

- @ComponentScan 은 @Component 가 붙은 모든 클래스를 스프링 빈으로 등록한다.
이때 스프링 빈의 **기본 이름은 클래스명을 사용하되 맨 앞글자만 소문자를 사용한다.**
    - **빈 이름 기본 전략:** `MemberServiceImpl` 클래스 → `memberServiceImpl`
    - **빈 이름 직접 지정:** 만약 스프링 빈의 이름을 직접 지정하고 싶으면`@Component("이름")`

### 2. @Autowired 의존관계 자동 주입

![Untitled](/assets/images/post/2023-08-09/2.png)

- 생성자에 `@Autowired` 를 지정하면, 스프링 컨테이너가 자동으로 해당 스프링 빈을 찾아서 주입한다.
    - 이때 기본 조회 전략은 **타입이 같은 빈을 찾아서 주입**한다.
- `getBean(MemberRepository.class)` 와 동일하다고 이해하면 된다.

## 탐색 위치와 기본 스캔 대상

### 탐색할 패키지의 시작 위치 지정

```java
@ComponentScan(
		basePackages = "raindragonn.core",
)
```

- 모든 자바 클래스를 다 컴포넌트 스캔하면 시간이 오래 걸린다.
    - 꼭 필요한 위치부터 탐색하도록 시작위치를 지정할 수 있다.
- `basePackages` : 탐색할 패키지의 시작 위치를 지정한다.
    - `basePackages = { “raindragonn.core”, “raindragonn.service” }`
        - 여러 시작 위치를 지정할 수도있다.

### 권장 방법

- 패키지 위치를 지정하지 않고, 설정 정보 클래스의 위치를 **프로젝트 최상단**에 두는 것이다. 최근 스프링 부트도 이 방법을 기본으로 제공한다.
    - `basePackages` 를 지정하지 않으면 기본적으로 해당 설정정보의 위치를 기반으로 한다.

### 컴포넌트 스캔 기본 대상

- `@Component` : 컴포넌트 스캔에서 사용
- `@Controller` : 스프링 MVC 컨트롤러에서 사용
    - 스프링 MVC 컨트롤러로 인식
- `@Service` : 스프링 비즈니스 로직에서 사용
    - `@Service` 는 특별한 처리를 하지 않는다. 대신 개발자들이 핵심 비즈니스 로직이 여기에
    있겠구나 라고 비즈니스 계층을 인식하는데 도움이 된다.
- `@Repository` : 스프링 데이터 접근 계층에서 사용
    - 스프링 데이터 접근 계층으로 인식하고, 데이터 계층의 예외를 스프링 예외로 변환해준다.
- `@Configuration` : 스프링 설정 정보에서 사용
    - 스프링 설정 정보로 인식하고, 스프링 빈이 싱글톤을 유지하도록 추가 처리를 한다.

**[ 위 어노테이션들이 `@Component`를 포함하고있다. ]** 

- 어노테이션에는 상속관계라는 것이 없다. 그래서 이렇게 애노테이션이 특정 애노테이션을 들고있는 것을 인식할 수 있는 것은 자바 언어가 지원하는 기능은 아니고, 스프링이 지원하는 기능이다.

## 필터

### includeFilters

- 컴포넌트 스캔 대상을 추가로 지정한다.

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface MyIncludeComponent {}

@MyIncludeComponent
public class BeanA {}

public class ComponentFilterAppConfigTest {
    @Test
    void filterScan() {
        ApplicationContext ac = new
AnnotationConfigApplicationContext(ComponentFilterAppConfig.class);
        BeanA beanA = ac.getBean("beanA", BeanA.class);
        assertThat(beanA).isNotNull();
}
    @Configuration
    @ComponentScan(
            includeFilters = @Filter(type = FilterType.ANNOTATION, classes = MyIncludeComponent.class)
    )
    static class ComponentFilterAppConfig {
    }
}
```

### excludeFIlters

- 컴포넌트 스캔에서 제외할 대상을 지정한다.

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface MyExcludeComponent {}

@MyIncludeComponent
public class BeanB {}

public class ComponentFilterAppConfigTest {
    @Test
    void filterScan() {
        ApplicationContext ac = new
AnnotationConfigApplicationContext(ComponentFilterAppConfig.class);
				assertThrows(
                NoSuchBeanDefinitionException.class,
                () -> ac.getBean("beanB", BeanB.class)
        );
}
    @Configuration
    @ComponentScan(
            excludeFilters = @Filter(type = FilterType.ANNOTATION, classes = MyExcludeComponent.class)
    )
    static class ComponentFilterAppConfig {

    }
}
```

**[ 각 어노테이션을 만듬으로써 컴포넌트 스캔에 추가 및 생략을 지정할 수 있다.]** 

### FilterType 옵션

- ANNOTATION: 기본값, 애노테이션을 인식해서 동작한다.
ex) org.example.SomeAnnotation
- ASSIGNABLE_TYPE: 지정한 타입과 자식 타입을 인식해서 동작한다.
ex) org.example.SomeClass
- ASPECTJ: AspectJ 패턴 사용
ex) org.example..*Service+
- REGEX: 정규 표현식
ex) org\.example\.Default.*
- CUSTOM: TypeFilter 이라는 인터페이스를 구현해서 처리
ex) org.example.MyTypeFilter

## 중복 등록과 충돌

### 자동 빈 등록 vs 자동 빈 등록

- 컴포넌트 스캔에 의해 자동으로 스프링 빈이 등록되는데, 그 이름이 같은 경우 스프링은 오류를 발생한다.
    - `ConflictingBeanDefinitionException` 예외 발생

### 수동 빈 등록 vs 자동 빈 등록

- 수동 빈 등록이 우선권을 가진다.
    - 수동 빈이 자동 빈을 오버라이딩 해버린다.
    - **최근에는 이런 경우 스프링 부트에서 오류를 발생**
    

## 코드 확인 [[ Link ]](https://github.com/raindragonn/spring-basic-exam/pull/4) - 해당 pr의 커밋 확인