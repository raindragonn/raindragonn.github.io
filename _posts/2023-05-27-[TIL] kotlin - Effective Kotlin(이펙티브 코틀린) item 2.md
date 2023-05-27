---
layout: "post"
title: "[TIL] Kotlin - 변수의 스코프를 최소화하라"
subtitle: "Effective kotlin - Item 2"
date: 2023-05-27
author: "raindragonn"
lang: ko
categories: [TIL - Kotlin]
tags:
  - Kotlin
  - Effective Kotlin
  - 이펙티브 코틀린
---

## 변수의 스코프를 최소화하라

## 스코프를 최소화하는 것이 좋다.

- 프로퍼티보다는 지역변수를 사용하는 것이 좋다.
- 최대한 좁은 스코프를 갖도록 변수를 사용하는것이 좋다.
    - 반복문 내부에서만 변수가 사용되는 경우, 변수를 반복문 내부에 작성하는 것이 좋다.

<aside>
💡 요소의 **스코프란** 요소를 볼 수 있는 영역(visible)을 의미한다. (인식할수 없다면 사용할 수 없다.)

</aside>

```kotlin
val a = 1
fun fizz() {
		val b = 2
		priunt(a + b)
}
val buzz = {
		val c = 3
		print(a + c)
}
// 현재 위치에서는 a를 사용할 수 있지만, b와 c는 사용할 수 없다.
```

### 나쁜 예

```kotlin
var user: User
for (i in users.indices) {
		user = users[i]
		println("User at $i is $user")
}
```

- 변수 user의 스코프는 반복문 외부에서도 사용이 가능하다.
- 변수가 다른 개발자 혹은 다른 부분에 의해서 잘못 사용될 수 있다.

### 조금 더 좋은 예

```kotlin
for (i in users.indices) {
		val user = users[i]
		println("User at $i is $user")
}
```

- 변수 user를 사용하는 반복문에서만 사용할 수 있다.

### 제일 좋은 예

```kotlin
for ((i, user) in users.withIndex()) {
		println("User at $i is $user")
}
```

- 스코프를 좁게 만듬으로써 프로그램을 추적하고 관리하기 쉽다.
- 어느 시점에 어떤 요소가 있는지 알 수 있다.
- 변경의 추적이 쉽다.

## 변수를 정의할 때 초기화 되는 것이 좋다.

- if, when, try-catch, Elvis 표현식 등을 통해 변수를 정의할 때 초기화할 수 있다.

### 나쁜 예

```kotlin
val user: User
if (hasValue) {
		user = getValue()
} else {
		user = User()
}
```

### 조금 더 좋은 예

```kotlin
val user: User = if(hasValue) {
		getValue()
} else {
		User()
}
```

## 구조분해 선언 활용하기.

- 여러 프로퍼티를 한꺼번에 설정하는 경우에 구조분해 선언을 활용하는 것이 좋다.

### 나쁜 예

```kotlin
fun updateWeather(degrees: Int) {
		val description: String
		val color: Int
		if (degrees < 5) {
				description = "cold"
				color = Color.BLUE
		} else if (degrees < 23) {
				description = "mild"
				color = Color.YELLOW				
		} else {
				description = "hot"
				color = Color.RED				
		}
		///...
}
```

### 조금 더 좋은 예

```kotlin
fun updateWeather(degrees: Int) {
		val (description, color = when {
				degrees < 5 -> "cold" to Color.BLUE
				degrees < 23 -> "mild" to Color.YELLOW
				else -> "hot" to Color.RED
		}
}
```

## 변수의 스코프가 넓으면 위험한 점.

### 람다 캡처링 문제

- 람다식 외부에서 선언한 변수를 람다식 내부에서 사용하는 경우 발생.
- 가변성을 피하고 스코프 범위를 좁게 만들면 ,이러한 문제를 피할 수 있다.
- [**[ link ]**](https://velog.io/@ams770/kotlin-lambda-and-capture-value)

## 정리

- 변수의 스코프는 좁게 만들어서 사용하는 것이 좋다.
- var보다는 val를 사용하는 것이 좋다.
- 람다에서는 변수를 캡처하는 문제가 있다.