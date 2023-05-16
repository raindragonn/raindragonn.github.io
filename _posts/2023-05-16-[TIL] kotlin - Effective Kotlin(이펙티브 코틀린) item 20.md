---
layout: "post"
title: "[TIL] Kotlin - 일반적인 알고리즘을 반복해서 구현하지 말라."
subtitle: "Effective kotlin - Item 20"
date: 2023-05-16
author: "raindragonn"
lang: ko
categories: [TIL - Kotlin]
tags:
    - Kotlin
    - Effective Kotlin
    - 이펙티브 코틀린
---

## 일반적인 알고리즘을 반복해서 구현하지 말라

## 들어가기에 앞서..

- 이번 아이템에서 다뤄볼 알고리즘이란 특정 프로젝트에 국한된 것이아니라, 수학적인 연산, 수집 처리와 같은 별도의 모듈 또는 라이브러리로 분리할 수 있는 부분을 의미합니다.
    - ex)숫자를 특정 범위에 맞추는 간단한 알고리즘.
    
    ```kotlin
    val percent = when {
    		numberFromUser > 100 -> 100
    		numberFromUser < 0 -> 0
    		else -> numberFromUser
    }
    ```
    
- 위와 같은 알고리즘은 stdlib의 `coreceIn` 확장 함수로 이미 존재합니다.[[ Link ]](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.ranges/coerce-in.html)

### 이미 만들어져있다!

- 위에서 본것처럼 이미 코틀린에서 지원하기에 따로 구현하지 않아도 되는 경우가 있습니다.
- 알고리즘을 작성하는 것보다 호출 한번이면 됩니다.
- 함수의 이름, 파라미터 등을 통해 무엇을 하는지 확실하게 알 수 있습니다.
    
    <aside>
    💡 개인적인 팁으로는 Intellij를 사용하는 경우 F1, cmd+b 를 통해 구현부를 확인
    
    </aside>
    
- 직접 구현하면서 발생할 수 있는 실수를 줄일 수 있습니다.
- 코틀린 팀에서 최적화하면 이러한 함수를 활용하는 모든 곳에서 혜택을 받을 수 있습니다.

## 표준 라이브러리 살펴보기

- 일반적인 알고리즘은 대부분 이미 다른 사람이 정의해 놓았습니다.
- 대표적인 라이브러리는 표준 라이브러리인 **`stdlib`**
    - 확장 함수를 활용해서 만들어진 거대한 유틸 라이브러리
    - [[ Link ]](https://kotlinlang.org/api/latest/jvm/stdlib/)

### 활용하기.

- 아래는 서버에서 결과값을 받아서 Database에 저장하는 예시입니다.

```kotlin
fun saveCallResult(response: SourceResponse) {
		var entityList = ArrayList<SourceEntity>()
		response.sources.forEach {
				var entity = SourceEntity()
				entity.id = it.id
				entity.category = it.category
				entity.country = it.country
			...
				entityList.add(entity)
		}
		db.insertSources(entityList)
}
```

- 위 코드에서 forEach를 사용하는 것보다 `Collection.map()` 을 사용하는게 더 좋습니다.

```kotlin
fun saveCallResult(response: SourceResponse) {
		var entityList = response.sources.map {
				SourceEntity(
						it.id,
						it.category,
						it.country,
					...
				)
		}
		db.insertSources(entityList)
}
```

## 나만의 유틸리티 구현하기.

### 언제 만들면 좋을까?

- 표준 라이브러리에 없는 알고리즘이 필요한 경우
    - ex) 컬렉션에 있는 모든 숫자의 곱을 계산하는 기능이 필요한 경우 (fold 구현부 [[ Link ]](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.sequences/fold.html) )

```kotlin
fun Iterable<Int>.product() =
		fold(1) { acc, i -> acc * i } 
```

### 주의할 점

1. 여러 번 사용되지 않는다고 해도 범용 유틸리티 함수로 정의 하는 것이 좋다. (확장 함수)
2. 가능하면 대부분의 개발자가 **이름을 통해 결과를 예측할 수 있도록 해야한다.**
3. 동일한 결과를 얻는 함수를 여러 번 만들지 말라.
    - 추가적인 리소스낭비가 없도록하기 위해 (테스트, 유지보수 등)
    - 기존에 관련된 함수가 있는지 탐색하는 과정도 필요하다.

### 확장 함수로 구현 시 장점

톱레벨 함수, 프로퍼티 위임, 클래스 등의 다양한 방법들과 비교시 장점으로

1. 함수는 상태를 유지하지 않기에, 행위를 나타내기 좋다.
2. 톱레벨 함수와 비교해서, 확장 함수는 구체적인 타입이 있는 객체에만 사용을 제한할 수 있다.
3. 수정할 객체를 아규먼트로 전달받아 사용하는 것보다는 확장 리시버로 사용하는 것이 가독성이 좋다.
4. 자동 완성 기능등으로 인해 쉽게 찾아서 사용할 수 있다.
    - 특정 Util, Helper클래스 등을 통해서 사용하는 경우 이러한 기능이 해당 클래스에 포함되어 있는지를 알고 있어야한다.

## 정리

일반적인 알고리즘의 경우 대부분 stdlib에 이미 정의되어 있을 가능성이 높습니다. 따라서 stdlib과 같은 kotlin에서 기본적으로 제공하는 다양한 라이브러리들을 공부해 두면 좋습니다.

제공하지 않는 알고리즘이 필요한 경우 프로젝트 내부에 직접 정의하고, 되도록이면 확장 함수로 정의합시다~!