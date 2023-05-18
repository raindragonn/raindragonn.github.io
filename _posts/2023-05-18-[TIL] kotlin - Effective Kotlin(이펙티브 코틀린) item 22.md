---
layout: "post"
title: "[TIL] Kotlin - 일반적인 알고리즘을 구현할 때 제네릭을 사용하라."
subtitle: "Effective kotlin - Item 22"
date: 2023-05-18
author: "raindragonn"
lang: ko
categories: [TIL - Kotlin]
tags:
    - Kotlin
    - Effective Kotlin
    - 이펙티브 코틀린
---

## 일반적인 알고리즘을 구현할 때 제네릭을 사용하라.

## 제네릭의 이점

```kotlin
inline fun <T> Iterable<T>.filter(
		predicate: (T) -> Boolean
): List<T> {
		val destination = ArrayList<T>()
		for (element in this) {
				if(predicate(element)) {
						destination.add(element)
				}
		}
		return destination
}
```

- 위 예제는 타입 파라미터 T를 갖는 제네릭 함수
- 타입 파라미터는 컴파일러에 타입과 관련된 정보를 제공하여 컴파일러가 타입을 더 정확하게 추측할 수 있도록 도와줍니다.
- IDE가 이를 기반으로 유용한 제안을 한다.
    - T의 타입이 T타입임을 강제

## 제네릭 제한

```kotlin
fun <T: Comparable<T>> Iterable<T>.sorted(): List<T> {
...
}

fun <T, C: MutableCollection<in T>>
Iterable<T>.toCollection(destination: C): C {
...
}

class ListAdapter<T: ItemAdapter>(...) { ...}
```

- 구체적인 타입의 서브타입만 사용하도록 타입을 제한하는것이 가능하다.
    - 즉, 특정 Class의 Sub Class만 사용하도록
- 타입에 제한을 걸음으로써 내부에서 해당 타입이 제공하는 메서드를 사용할 수 있다.

## 제한으로 Any를 많이 사용한다

```kotlin
inline fun <T, : Any> Iterable<T>.mapNotNull(
		trasnform: (T) -> R?
): List<R> {
		return mapNotNullTo(ArrayList<R>(), transform)
}
```

- Any로 타입을 제한한 경우 이는 `nullable` 이 아님을 나타냅니다.

![스크린샷 2023-05-17 오후 12.11.45.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c7a80ea9-8582-4f7f-b65b-f11d434e9ec2/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2023-05-17_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_12.11.45.png)

## 정리

- 제네릭은 타입 제한을 통해 특정 자료형이 제공하는 메서드를 안전하게 사용할 수 있다.