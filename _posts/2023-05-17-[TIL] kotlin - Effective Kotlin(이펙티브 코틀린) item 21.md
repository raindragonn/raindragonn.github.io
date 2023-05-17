---
layout: "post"
title: "[TIL] Kotlin - 일반적인 프로퍼티 패턴은 프로퍼티 위임으로 만들어라."
subtitle: "Effective kotlin - Item 21"
date: 2023-05-17
author: "raindragonn"
lang: ko
categories: [TIL - Kotlin]
tags:
    - Kotlin
    - Effective Kotlin
    - 이펙티브 코틀린
---

## 일반적인 프로퍼티 패턴은 프로퍼티 위임으로 만들어라.

## 들어가기에 앞서..

- 코틀린에서는 코드 재사용과 관련해서 프로퍼티 위임이란느 기능을 제공합니다.
- 대표적인 예시가 처음 사용하는 요청이 들어올 때 초기화 되는 지연 프로퍼티인 `lazy`
- 자바 등에서는 어노테이션을 많이 활용하지만 코틀린에서는 프로퍼티 위임을 통해 다양한 패턴들을 쉽게 만들 수 있습니다.

## Delegates.observable

- [stdlib](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.properties/-delegates/observable.html)에서 제공
- 변화를 감지하는 observable 패턴을 쉽게 만들 수 있다.
    
    ```kotlin
    var observed = false
    var max: Int by Delegates.observable(0) { _, _, _ ->
        observed = true
    }
    
    println(max) // 0
    println("observed is ${observed}") // false
    
    max = 10
    println(max) // 10
    println("observed is ${observed}") // true
    ```
    

## Type-safe하게 구현이 가능하다.

```kotlin
// 안드로이드에서의 뷰와 리소스 바인딩
private val button: Button by bindView(R.id.button)
private val textSize by bindDimension(R.climen.font_size)
private val doctor: Doctor by argExtra(DOCTOR_ARG)

// Koin에서의 종속성 주입
private val presenter: MainPresenter by inject()
private val repository: NetworkRepository by inject()
private val vm: MainViewModet by viewModel()

// 데이터 바인딩
private val port by bindConfiguration("port")
private val token: String by preferences.bind(TOKEN_KEY)
```

## 프로퍼티 위임을 사용하지 않는 경우

- 일부 프로퍼티가 사용될 때, 로그를 함께 출력하고 싶은 경우

### 게터와 세터를 이용한다.

```kotlin
var token: String? = null
		get() {
				print("token값은 $field입니다.")
				return field
		}
		set(value) {
				print("token값이 $field에서$value로 변경되었습니다.")
				field = value
		}
```

## 프로퍼티 위임을 활용하는 방법

- 다른 객체의 메서드를 활용해서 프로퍼티의 접근자를 만드는 방식
    - 쉽게 말해 대리자를 통해 게터, 세터를 만든느 방식을 말한다.
- 게터는 `getValue`, 세터는 `setValue` 라는 operater function을 만들어야 합니다.
- val의 경우 `getValue`, var의 경우 `getValue`, `setValue` 를 필요로한다.

```kotlin
class LoggingProperty<T>(var value: T) {
		operator fun getValue(
				thisRef: Any?,
				prop: KProperty<*>
		): T {
				print("${prop.name}의 값은 $value입니다.")
				return value
		}
		
		operator fun setValue(
				thisRef: Any?,
				prop: KProperty<*>,
				newValue: T
		) {
				val name = prop.name
				print("$name값이 $value에서 $newValue로 변경되었습니다.")
				value = newValue
		}
}

var token: String? by LoggingProperty(null)
var attempts: Int by LoggingProperty(0)
```

> 위 코드중 token은 어떻게 컴파일 될까?
> 

```kotlin
@JvmField
private val 'token$delegate' =
		LoggingProperty<String?>(null)

var token: String?
		get() = 'token$delegate'.getVaXue(this, ::token)
		set(value) {
				'token$delegate'.setValue(this, ::token, value)
		}
```

1. 단순히 값만 처리하도록 변경되는 것이 아니라 컨텍스트와 프로퍼티 레퍼런스의 경계도 함께 사용하는 형태로 사용된다.
2. 프로퍼티에 대한 레퍼런스는 이름, 어노테이션과 관련된 정보등을 얻을 때 사용된다.
3. 컨텍스트는 함수가 어떤 위치에서 사용되는지와 관련된 정보를 제공해 준다.

### 다양한 타입을 지원하는 델리게이트

<aside>
💡 getValue와 serValue 메서드가  여러 개 있어도 컨텍스트를 활용하므로 상황에 따라 적절한 메서드가 선택된다.

아래 예시는 Android에서 사용하는 Activity, Fragment 두 곳 모두에서 사용이 가능하다.

</aside>

```kotlin
class SwipeRefreshBinderDelegate(val id: Int) {
		private var cache: SwipeRefreshLayout? = null

		operator fun getValue(
				activity: Activity,
				prop: KProperty<*>
		): SwipeRefreshLayout {
				return cache ?: activity
						.findViewById<SwipeRefreshLayout>(id)
						.also { cache = it }
		}

		operator fun getValue(
				fragment: Fragment,
				 prop: KProperty<*>
		): SwipeRefreshLayout {
				return cache ?: fragment.view
				.findViewById<SwipeRefreshLayout>(id)
				.also { cache = it }
		}
}
```

### 확장함수를 통해 만드는 방법

- Map<String, *>를 사용하는 예시

```kotlin
val map: Map<String, Any> = mapOf(
		"name" to "YongWoo",
		"kotlinProgrammer" to true
)
val name by map
println(name) // YongWoo
```

- 위와 같은 방식이 가능한 이유는 stdlib에 아래와 같은 확장 함수가 정의되어 있어서 가능하다.

```kotlin
inline operator fun <V, V1 : V> Map<in String, V>
.getValue(thisRef: Any?, property: KProperty<*>): V1 = 
getOrImplicitDefault(property.name) as V1
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/91ad1716-b738-466e-bc3d-3f4eb12b7f36/Untitled.png)

### stdlib에서 지원하는 프로퍼티 델리게이터

- lazy
- Delegates.observable
- Delegats.vetoable
- Delegates.notNull

## 정리

- 프로퍼티 델리게이트를 이용하면 프로퍼티와 관련된 다양한 추가 조작이 가능하다.
- 프로퍼티 위임을 통해 다양하게 재사용이 가능하다.
- 프로퍼티 위임이라는 강력한 도구를 활용한다면 일반적인 패턴을 재사용하거나 더 좋은 API를 만들 떄 활용할 수 있다.