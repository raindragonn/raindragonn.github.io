---
layout: "post"
title: "[TIL] Kotlin - 예외를 활용해 코드에 제한을 걸어라"
subtitle: "Effective kotlin - Item 5"
date: 2023-06-22
author: "raindragonn"
lang: ko
categories: [TIL - Kotlin]
tags:
  - Kotlin
  - Effective Kotlin
  - 이펙티브 코틀린
---

## 예외를 활용해 코드에 제한을 걸어라

## 확실한 동작을 원하는 경우 예외를 걸어라.

### 코틀린에서 코드의 동작에 제한을 거는 방법

1. `require` 블록: 아규먼트를 제한할 수 있다.
2. `check` 블록: 상태와 관련된 동작을 제한할 수 있다.
3. `assert` 블록: 어떤 것이 true 인지 확인할 수 있다.(테스트 코드에서만 사용)
4. `Elvis 연산자` : return 또는 throw와 함께 활용하여 사용한다.

아래는 Stack<T>의 일부

```kotlin
fun pop(num: Int = 1): List<T> {
		require(num <= size) {
				"Cannot remove more elements than current size"
		}
		check(isOpen) { "Cannot pop from closed Stack" }
		val ret = collection.take(num)
		collection = collection.drop(num)
		assert(ret.size == num)
		return ret
}
```

위와 같은 제한을 걸어주는 경우 장점이 있다.

- 제한을 걸어 문서를 읽지 않고 코드만으로 문제를 확인할 수 있다.
- 문제가 있을 경우에 예상하지 못하는 동작이 아닌 예외를 throw하여 보다 안전하게 상태를 관리할 수 있다.
- 카드가 어느정도 자체적으로 검사된다. 그에따라, 단위 테스트를 줄일 수 있다.
- 스마트 캐스트 기능을 활용할 수 있게 되므로, 타입 변환을 적게 할 수 있다.

## 아규먼트 - require

함수를 정의할 때 타입 시스템을 활용해서 아규먼트(argument)에 제한을 거는 코드를 많이 사용한다.

일반적으로 `require` 함수를 사용한다. `require` 함수는 제한을 확인하고, 제한을 만족하지 못한 경우 예외를 throw한다.

예시

```kotlin
fun factortial(n: Int) Long {
		require(n >= 0)
		return if (n <= 1) 1 else factorial(n - 1) * n
}
```

`require` 함수는 조건을 만족하지 못할 때 무조건적으로 `IllegalArgumentException` 을 발생시킨다.

일반적으로 이러한 처리는 함수의 가장 앞부분에서 하게 되므로 쉽게 확인할 수 있다.

## 상태 - check

어떤 구체적인 조건을 만족할 때만 함수를 사용할 수 있도록 해야하는 경우가 있다.

- 어떤 객체가 미리 초기화되어 있어야만 처리를 하게 하고 싶은 함수
- 사용자가 로그인했을 때만 처리를 하게 하고 싶은 함수
- 객체를 사용할 수 있는 시점에 사용하고 싶은 함수

`check` 함수는 `require` 와 비슷하지만, 지정된 예측을 만족하지 못할 때, `IllegalStateException` 을 throw 합니다.

**상태가 올바른지 확인할 때 사용한다.**

사용자가 규약을 어기고, 사용하면 안되는 곳에서 함수를 호출하고 있다고 의심될 때, 함수 전체에 대한 어떤 예측이 있을 때는 일반적으로 `require` 블록뒤에 배치한다.

## Assert 계열 함수 사용

### 단위 테스트

단위 테스트는 구현의 정확성을 확인하는 가장 기본적인 방법이다.

## nullability와 스마트 캐스팅

코틀린에서 require와 check 블록으로 어떤 조건을 확인해서 true가 나온경우, 해당 조건은 이후에도 true일 거라고 가정한다.

`Elvis`  연산자를 활용해 throw, return하는 경우 읽기 쉽고, 유연하게 사용할 수 있다.

## 정리

예외를 활용해 코드에 제한을 거는 경우 아래와 같은 이점이 있다.

- 제한을 훨씬 더 쉽게 확인할 수 있다.
- 애플리케이션을 더 안정적으로 지킬 수 있다.
- 코드를 잘못 쓰는 상황을 막을 수 있다.
- 스마트 캐스팅을 활용할 수 있다.