---
layout: "post"
title: "[TIL] Kotlin - inferred 타입으로 리턴하지 말라(타입 추론)"
subtitle: "Effective kotlin - Item 4"
date: 2023-06-20
author: "raindragonn"
lang: ko
categories: [TIL - Kotlin]
tags:
  - Kotlin
  - Effective Kotlin
  - 이펙티브 코틀린
---

## inferred 타입으로 리턴하지 말라.

## 들어가기에 앞서..

타입 추론(type inference)은 널리 알려진 코틀린의 특징이다.

자바는 자바10 부터 코틀린을 따라 타입 추론을 도입(코틀린과 비교 시 몇 가지 제약)

## 타입 추론을 사용할 때 주의할 점

### 특징

1. 할당 때 inferred 타입은 정확하게 오른쪽에 있는 피연산자에 맞게 설정된다
2. 슈퍼클래스 또는 인터페이스로는 설정되지 않는다.

```kotlin
open class Animal
class Zebra: Animal()

fun main() {
		var animal = Zebra()
		animal = Animal() // Type mismatch
}
```

해결법

```kotlin
open class Animal
class Zebra: Animal()

fun main() {
		var animal: Animal = Zebra()
		animal = Animal()
}
```

위와 같은 방법으로 문제를 해결할 수 있지만, 직접 코드를 조작할 수 없는 경우(작성 권한이 없는 라이브러리 또는 모듈)는 이러한 문제를 해결할 수 없다.

### 문제 예시

아래와 같은 인터페이스와 디폴트 값을 타입 추론으로 사용할 때

```kotlin
val DEFAULT_CAR: Car = Avante()

interface CarFactory {
		fun produce() = DEFAULT_CAR
}
```

이후 다른 사람이 DEFAULT_CAR는 타입 추론에 의해 자동으로 타입이 지정될 것이므로, Car를 명시적으로 지정하지 않아도 된다고 생각해서 다음과 같이 코드를 변경하는 경우

```kotlin
val DEFAULT_CAR = Avante()
```

`CarFactory` 에서 이제 `Avante` 이외의 자동차를 생산하지 못한다.

## 정리

- **타입을 확실하게 지정해야 하는 경우에는 명시적으로 타입을 지정해야 한다는 원칙을 지켜라.**
    - 굉장히 중요한 정보이므로, 숨기지 않는것이 좋다.
- 외부 API를 만드는 경우 반드시 타입을 지정하며, 확실한 확인 없이 제거하지 말라.
- inferred 타입은 프로젝트가 커지면서 제한이 많아지거나 예측하지 못한 결과를 낼 수 있다.