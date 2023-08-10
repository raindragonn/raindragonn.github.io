---
layout: "post"
title: "Spring 핵심 원리 - 의존관계 자동 주입"
subtitle: "Spring 핵심 원리 - 의존관계 자동 주입"
date: 2023-08-10
author: "raindragonn"
lang: ko
categories: [Spring]
tags:
  - Spring
  - Spring Boot
  - 기본
---

## 다양한 의존관계 주입 방법

### 1. 생성자 주입

- 이름 그대로 생성자를 통해서 의존 관계를 주입 받는 방법이다.

**[ 특징 ]**

- 생성자 호출 시점에 딱 1번만 호출되는 것이 보장된다.
- **불변**, **필수** 의존관계에 사용된다.
    - 외부에서 주입된 인스턴스에 대한 변경이 불가능 하도록 한다.
- **생성자가 1개만 있는 경우에 `@Autowired`를 생략해도 스프링에서 자동 주입해준다.**

```java
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

### 2. 수정자 주입 (setter 주입)

- setter라 불리는 필드의 값을 변경하는 수정자 메서드를 통해서 의존관계를 주입하는 방법이다.

**[ 특징 ]**

- **선택**, **변경** 가능성이 있는 의존관계에 사용
- 자바빈 프로퍼티 규약의 수정자 메서드 방식을 사용하는 방법이다.
- `@Autowired`의 기본 동작은 주입할 대상이 없으면 오류가 발생한다.
    - 주입할 대상이 없어도 동작하게 하려면 `@Autowired(required = false)`로 지정하면 된다.

```java
@Component
public class OrderServiceImpl implements OrderService {

    private MemberRepository memberRepository;
    private DiscountPolicy discountPolicy;

    @Autowired
		public void setMemberRepository(MemberRepository memberRepository) {
				this.memberRepository = memberRepository;
		}

		@Autowired
    public void setDiscountPolicy(DiscountPolicy discountPolicy) {
        this.discountPolicy = discountPolicy;
    }
		...
}
```

### 3. 필드 주입

- 이름 그대로 필드에 바로 주입하는 방법이다.

**[ 특징 ]**

- 코드가 간결하지만 외부에서 변경이 불가능해서 테스트하기 힘들다는 단점이 있다.
- DI 프레임워크가 없으면 아무것도 할 수 없다.
    - 순수한 자바 테스트 코드에서 `@Autowired` 는 동작하지 않는다. `@SpringBootTest` 처럼 스프링 컨테이너를 테스트에 통합한 경우에만 가능하다.
- **가급적 사용하지 않는다.**
    - 상용 코드와 관계 없는 테스트 코드에서 사용하거나
    - 스프링 설정을 목적으로 하는 `@Configuration` 같은 곳에서만 특별한 용도로 사용한다.

```java
@Component
public class OrderServiceImpl implements OrderService {

		@Autowired
    private MemberRepository memberRepository;
		@Autowired
    private DiscountPolicy discountPolicy;
		...
}
```

### 4. 일반 메서드 주입

- 일반 메서드를 통해서 주입 받을 수 있다.

**[ 특징 ]**

- 한번에 여러 필드를 주입 받을 수 있다.
- **일반적으로 잘 사용하지 않는다.**

```java
@Component
public class OrderServiceImpl implements OrderService {

    private MemberRepository memberRepository;
    private DiscountPolicy discountPolicy;

		@Autowired
		public void init(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
				this.memberRepository = memberRepository;
		}

		...
}
```

## 옵션 처리

- 주입할 스프링 빈이 없어도 동작해야 할 때가 있다.
- `@Autowired`의 `required` 옵션의 기본값이 `true` 로 되어 있어서 자동 주입 대상이 없으면 오류가 발생한다.
    - `@Autowired(required=false)` : 자동 주입할 대상이 없으면 수정자 메서드 자체가 호출 안됨
    - `org.springframework.lang.@Nullable` : 자동 주입할 대상이 없으면 null이 입력된다.
    - `Optional<>` : 자동 주입할 대상이 없으면 Optional.empty 가 입력된다.

```java
public class AutowiredTest {

    @Test
    void autowiredOption() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(TestBean.class);
    }

    static class TestBean {

        @Autowired(required = false)
        public void setNoBean1(Member member) {
            System.out.println("noBean1 = " + member);
        }

        @Autowired
        public void setNoBean2(@Nullable  Member member) {
            System.out.println("noBean2 = " + member);
        }

        @Autowired(required = false)
        public void setNoBean3(Optional<Member> optionalMember) {
            System.out.println("noBean3 = " + optionalMember);
        }
    }
}
--------------로그---------------
noBean2 = null
noBean3 = Optional.empty
```

- Member는 스프링 빈으로 등록한 적이 없다.

## 생성자 주입을 사용해라

- 과거에는 수정자 주입과 필드 주입을 많이 사용했지만, 최근에는 스프링을 포함한 DI 프레임워크 대부분이 **생성자 주입을 권장**한다.

### 이유 1. 불변

- 대부분의 의존관계 주입은 한번 일어나면 애플리케이션 종료시점까지 의존관계를 변경할 일이 없다. 오히려
대부분의 의존관계는 애플리케이션 종료 전까지 변하면 안된다.(불변해야 한다.)
- 수정자 주입을 사용하면, setXxx 메서드를 public으로 열어두어야 한다.
    - 누군가 실수로 변경할 수 도 있고, 변경하면 안되는 메서드를 열어두는 것은 좋은 설계 방법이 아니다.
- 생성자 주입은 객체를 생성할 때 딱 1번만 호출되므로 이후에 호출되는 일이 없다. 따라서 불변하게 설계할
수 있다.

### 이유 2. 누락

- 프레임워크 없이 순수한 자바 코드를 단위 테스트 하는 경우 `@Autowired` 는 아무 동작이 없기에 의존관계 주입이 누락되는 문제가 발생한다.
    - 생성자 주입을 사용하는 경우에는 의존관계를 명확하게 확인할 수 있으며, 누락한 경우 **컴파일 오류가 발생**해 문제를 바로 확인할 수  있다.

### 이유 3. final 키워드

- 생성자 주입을 사용하면 필드에 final 키워드를 사용할 수 있다. 그래서 생성자에서 혹시라도 값이 설정되지 않는 오류를 컴파일 시점에 막아준다
    - `java: variable 의존관계 might not have been initialized`

### [ 정리 ]

- 생성자 주입 방식을 선택하는 이유는 여러가지가 있지만, 프레임워크에 의존하지 않고, 순수한 자바 언어의
특징을 잘 살리는 방법이기도 하다.
- 기본으로 생성자 주입을 사용하고, 필수 값이 아닌 경우에는 수정자 주입 방식을 옵션으로 부여하면 된다. 생
성자 주입과 수정자 주입을 동시에 사용할 수 있다.
- 항상 생성자 주입을 선택해라! 그리고 가끔 옵션이 필요하면 수정자 주입을 선택해라. 필드 주입은 사용하지
않는게 좋다.

## 롬복과 최신 트렌드

- 생성자 주입이 좋지만 보일러 플레이트 코드가 많이 생긴다. 이를 개선하기 위한 방법이다.

### 롬복이란?

- JAVA의 라이브러리 중 하나로 Annotation processing이라는 기술을 통해 어노테이션을 통해 반복되는 메소드를 자동으로 작성해준다.

### 롬복 사용하기

1. 롬복은 어노테이션 프로세서 기능을 사용한다. 그에따른 옵션 추가
    
    ```groovy
    configurations {
        compileOnly {
            extendsFrom annotationProcessor
        }
    }
    ```
    
2. 롬복 라이브러리 추가
    
    ```groovy
    dependencies {
    		...
    		compileOnly 'org.projectlombok:lombok'
    		annotationProcessor 'org.projectlombok:lombok'
    
    		testCompileOnly 'org.projectlombok:lombok'
    		testAnnotationProcessor 'org.projectlombok:lombok'
    		...
    }
    ```
    
3. 롬복 플러그인 추가 - intellij의 plugins에서 lombok을 추가한다.
4. settings → build, Execution, Deployment → Compiler → Annotation Processors → Enable Annotaion processing 켜기
    
    ![Untitled](/assets/images/post/2023-08-10/1.png)
    

### 최적화 방법

```java
@Component
public class OrderServiceImpl implements OrderService {

		private final MemberRepository memberRepository;
		private final DiscountPolicy discountPolicy;

		@Autowired
    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
				this.memberRepository = memberRepository;
				this.discountPolicy = discountPolicy;
    }
}
```

```java
@Component
@RequiredArgsConstructor
public class OrderServiceImpl implements OrderService {
		private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;
}
```

1. 생성자가 한개만 있는 경우 `@Autowired` 를 생략할 수 있다.
2. 롬복 라이브러리가 제공하는 `@RequredArgsConstructor` 어노테이션을 사용하면 final이 붙은 필드를 모아서 생성자를 자동으로 만들어준다.
    - 롬복이 자바의 어노테이션 프로세서라는 기능을 이용해서 컴파일 시점에 생성자 코드를 자동으로 생성해준다.

## 문제 상황 - 조회 빈이 2개 이상

- `@Autowired` 는 타입(Type)으로 조회한다.
    - 다음 코드와 유사하게 동작한다. (실제로는 더 많은 기능을 제공한다.) `ac.getBean(DiscountPolicy.class)`
- 타입으로 조회하는 경우 선택된 빈이 2개 이상일 때 문제가 발생한다.
    - `DisCountPolicy` 의 하위 타입 `FixDiscountPolicy`, `RateDiscountPolicy` 두 가지가 스프링 빈으로 선언한 경우
    - `NoUniqueBeanDefinitionException` 오류가 발생한다.
- 하위 타입으로 지정할 수 도 있지만, **하위 타입으로 지정하는 것은 DIP를 위배하고 유연성이 떨어진다.**
그리고 이름만 다르고, 완전히 똑같은 타입의 스프링 빈이 2개 있을 때 해결이 안된다.

## 문제 해결 - @Autowired 필드명, @Qualifier, @Primary

### 1. @Autowired 필드 명 매칭

- @Autowired는 타입 매칭을 시도하고, 이때 여러 빈이 있는경우 필드 이름, 파라미터 이름으로 빈 이름을 추가 매칭한다.
- **필드 명 매칭은 먼저 타입 매칭을 시도하고 그 결과에 여러 빈이 있을 때 추가로 동작하는 기능이다.**
    
    ```java
    // 기존
    @Autowired
    private DiscountPolicy discountPolicy
    
    // 필드 명을 빈 이름을 ㅗ변경
    @Autowired
    private DiscountPolicy rateDiscountPolicy
    ```
    

### 2. @Qualifier 사용

- 추가 구분자를 붙여주는 방법이다.
- 단점으로는 주입 받을 때 모든 코드에 `@Qualifier`를 붙여주어야 한다는 점이다.
- 주입 시 추가적인 방법을 제공하는 것이지 **빈 이름을 변경하는 것은 아니다.**

```java
@Component
@Qualifier("mainDiscountPolicy")
public class RateDiscountPolicy implements DiscountPolicy {}
-----
@Bean
@Qualifier("mainDiscountPolicy")
public DiscountPolicy discountPolicy() {
		return new ...
}
```

- 스프링 빈에 추가 구분자를 붙여준다.

```java
@Autowired
public OrderServiceImpl(
		MemberRepository memberRepository,
		@Qualifier("mainDiscountPolicy")
		DiscountPolicy discountPolicy,
) {
      this.memberRepository = memberRepository;
      this.discountPolicy = discountPolicy;
}
```

- 주입 시 `@Qualifier` 를 통해 특정 빈을 구분해서 주입이 가능하다.

### 3. @Primary 사용하기

- 우선순위를 정하는 방법이다.
- `@Primary` 가 우선권을 가진다.

```java
@Component
@Primary
public class RateDiscountPolicy implements DiscountPolicy {}

@Component
public class FixDiscountPolicy implements DiscountPolicy {}
```

- 위 상황에서는 `RateDiscountPolicy` 가 우선권을 가진다.

### @Primary, @Qualifier 활용

- 코드에서 자주 사용하는 메인 스프링 빈이 있고, 특별한 기능으로 가끔 사용하는 스프링 빈이 있는 경우, 메인 스프링 빈은 `@Primary` 를 적용해서 주입받는 곳에서 `@Qualifier` 지정 없이 편리하게 주입하고, 가끔 사용하는 스프링빈을 주입해야하는 경우에는 `@Qualifer` 를 지정해서 명시적으로 주입하는 방식이 코드를 깔끔하게 유지하도록 도와준다.

### 우선순위

- `@Primary` - 기본값 처럼 동작
- `@Qualifer` - 매우 상세하게 동작
- 스프링은 자동보다는 수동이, 넓은 범위의 선택권 보다는 좁은 범위의 선택원이 우선순위가 높다.
    - 즉, `@Qualifier` 가 우선권이 높다.

## 어노테이션 직접 만들기

- `@Qualifer(”mainDiscountPolicy”)` 와 같이 문자를 적으면 컴파일시 타입 체크가 안된다.(문자로 지정하기에)
- 어노테이션을 만들어서 보다 확실하게 할 수 있다.

```java
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER,
  ElementType.TYPE, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Qualifier("mainDiscountPolicy")
public @interface MainDiscountPolicy {
}

@Component
@MainDiscountPolicy
public class RateDiscountPolicy implements DiscountPolicy {}

//생성자 자동 주입
@Autowired
public OrderServiceImpl(
		MemberRepository memberRepository,
		@MainDiscountPolicy DiscountPolicy discountPolicy) {
		this.memberRepository = memberRepository;
		this.discountPolicy = discountPolicy;
}
```

## 조회한 빈이 모두 필요한 경우 - List, Map

- 의도적으로 해당 타입의 스프링 빈이 다 필요한 경우가 있다.
    - ex) 할인 서비스를 제공하는데, 클라이언트가 할인의 종류(rate, fix) 를 선택할 수 있는 경우
- 스프링을 사용하면 소위 말하는 전략 패턴을 간단하게 구현할 수 있다.

```java
public class AllBeanTest {

    @Test
    void findAllBean() {
        AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AutoAppConfig.class, DiscountService.class);

        DiscountService discountService = ac.getBean(DiscountService.class);

        Member member = new Member(1L, "userA", Grade.VIP);
        int discountPrice = discountService.discount(member, 10000, "fixDiscountPolicy");

        assertThat(discountService).isInstanceOf(DiscountService.class);
        assertThat(discountPrice).isEqualTo(1000);

        int rateDiscountPrice = discountService.discount(member, 20000, "rateDiscountPolicy");
        assertThat(rateDiscountPrice).isEqualTo(2000);
    }

    static class DiscountService {

        private final Map<String, DiscountPolicy> policyMap;
        private final List<DiscountPolicy> policies;

        @Autowired
        public DiscountService(Map<String, DiscountPolicy> policyMap, List<DiscountPolicy> policies) {
            this.policyMap = policyMap;
            this.policies = policies;

            System.out.println("policyMap = " + policyMap);
            System.out.println("policies = " + policies);
        }

        public int discount(Member member, int price, String discountCode) {
            DiscountPolicy discountPolicy = policyMap.get(discountCode);
            return discountPolicy.discount(member, price);
        }
    }
}
```

### 로직 분석

- `DiscountService`는 `Map`으로 모든 `DiscountPolicy`를 주입받는다.
- `discount()` 메서드는 `discountCode`를 통해 주입받은 할인정책을 찾아서 실행한다.

### 주입 분석

- Map의 키에 스프링 빈의 이름을 넣어주고, 그 값으로 `DiscountPolicy` 타입으로 조회한 모든 스프링 빈을 담아준다.
- List의 제네릭 타입 `DiscountPolicy` 타입으로 조회한 모든 스프링 빈을 담아준다.
- 만약 해당하는 타입의 스프링 빈이 없는경우, 빈 컬렉션이나 Map을 주입한다.

### 참고 - 스프링 컨테이너를 생성하면서 스프링 빈 등록하기.

- 스프링 컨테이너는 생성자에 클래스 정보를 받으며 해당 클래스가 스프링 빈으로 자동 등록된다.
    - ex) `new AnnotationConfigApplicationContext(AutoAppConfig.class, DiscountService.class);` → `AutoAppConfig`, `DiscountService`

## 자동, 수동의 올바른 실무 운영 기준

### 편리한 자동 기능을 기본으로 사용하자.

- 스프링은 계층에 맞추어 애플리케이션 로직을 자동으로 스캔할 수 있도록 지원한다.
    - ex) `@Controller`, `@Service`, `@Repository`
    - 설정 정보를 기반으로 애플리케이션을 구성하는 부분과 실제 동작하는 부분을 명확하게 나누는 것이 이상적 이지만 과정이 상당히 번거롭다.

### 수동 빈 등록을 사용하는 경우

**[ 업무 로직 빈 ]**

- 웹을 지원하는 컨트롤러, 핵심 비즈니스 로직이 있는 서비스, 데이터 계층의 로직을 처리하는 리포지토리등이 모두 업무 로직이다. 보통 비즈니스 요구사항을 개발할 때 추가되거나 변경된다.
- **자동 기능을 적극 사용하는 것이 좋다.**
    - 업무 로직은 숫자도 매우 많고, 한번 개발해야 하면 컨트롤러, 서비스, 리포지토리 처럼 어느정도 유사한 패 턴이 있다.

**[ 기술 지원 빈 ]**

- 기술적인 문제나 공통 관심사(AOP)를 처리할 때 주로 사용된다. 데이터베이스 연결이나, 공 통 로그 처리 처럼 업무 로직을 지원하기 위한 하부 기술이나 공통 기술들이다.
- **가급적 수동 빈 등록을 사용해서 명확하게 드러내는 것이 좋다.**
    - 업무 로직과 비교해서 그 수가 매우 적고, 보통 애플리케이션 전반에 걸쳐서 광범위하게 영 향을 미친다.

> **애플리케이션에 광범위하게 영향을 미치는 기술 지원 객체는 수동 빈으로 등록해서 설정 정보에 바로 나
타나게 하는 것이 유지보수 하기 좋다.**
> 

### 비즈니스 로직 중에서 다형성을 적극 활용할 때 - 수동 빈 등록

- 자동 빈 등록을 사용한 경우 조회한 빈이 모두 필요할 때 어떤 빈들이 주입될 지, 각 빈들의 이름은 무엇일지 코드만 보고 한번에 파악하기가 쉽지않다.
    - 자동 등록을 사용한 경우 파악하기위해 여러 코드를 직접 찾아봐야 한다.
    - 이러한 경우 **별도의 설정 정보로 만들어서 수동으로 등록**하거나
    - 자동으로하는 경우 **특정 패키지에 같이 묶어**두는게 좋다.

```java
@Configuration
public class DiscountPolicyConfig {
		@Bean
		public DiscountPolicy rateDiscountPolicy() {
				return new RateDiscountPolicy();
    }
		@Bean
		public DiscountPolicy fixDiscountPolicy() {
				return new FixDiscountPolicy();
		}
}
```

### [ 정리 ]

- 자동 기능을 기본으로 사용하자.
- 직접 등록하는 기술 지원 객체는 수동 등록하자.
- 다형성을 적극 활용하는 비즈니스 로직은 수동 등록을 고민해보자.

## 코드 확인 [[ Link ]](https://github.com/raindragonn/spring-basic-exam/pull/5) - 해당 pr의 커밋내역 확인.