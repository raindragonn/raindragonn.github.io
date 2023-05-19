---
layout: "post"
title: "[TIL] Kotlin - 타입 파라미터의 새도잉을 피하라."
subtitle: "Effective kotlin - Item 23"
date: 2023-05-19
author: "raindragonn"
lang: ko
categories: [TIL - Kotlin]
tags:
    - Kotlin
    - Effective Kotlin
    - 이펙티브 코틀린
---

## 타입 파라미터의 새도잉을 피하라.

## 섀도잉이란?

```kotlin
class Programmer(
		private val name: String
) {
		fun addKeyboard(name: String){
			...
		}
}
```

- 위 예시 처럼 프로퍼티(name)와 파라미터(name)가 동일한 이름을 사용하는 경우 **지역 파라미터가 외부 스코프에 있는 프로퍼티를 가리는 문제**를 말한다.

```kotlin
interface Keyboard

class QK: Keyboard
class KeyChrone: Keyboard

class Programmer<T: Keyboard> {	
		fun <T: Keyboard> addKeyboard(keyboard: T) {
			...
		}
}

val programmer = Programmer<QK>()
programmer.addKeyboard(QK())
programmer.addKeyboard(KeyChrone())

```

- 위 예시처럼 클래스 타입 파라미터와 함수 타입 파라미터 사이에서도 발생하며 두 타입은 독립적으로 동작한다.
- 이러한 경우 개발자가 의도하는 경우가 거의 없으며, 문제를 찾아내기가 힘들어짐.

## 개선

```kotlin
class Programmer<T: Keyboard> {	
1>	fun addKeyboard(keyboard: T) {
			...
		}
		

2>	fun <ST: T> addKeyboard(keyboard: ST){ 
			...
		}
}

val programmer = Programmer<QK>()
programmer.addKeyboard(QK())
programmer.addKeyboard(KeyChrone()) // ERROR

```

1. 클래스 타입 파라미터를 사용하도록 변경
2. 독립적인 타입 파라미터를 사용하는 경우 이름을 다르게 사용.
    - 타입 파라미터를 사용해 다른 타입 파라미터에 제한을 주는것도 가능.

## 정리

- 코드를 읽고 이해하기 어려워지는 타입 파라미터 섀도잉을 피하자!