---
layout: "post"
title: "[TIL] Kotlin - 가변성을 제한하라"
subtitle: "Effective kotlin - Item 1"
date: 2023-05-26
author: "raindragonn"
lang: ko
categories: [TIL - Kotlin]
tags:
  - Kotlin
  - Effective Kotlin
  - 이펙티브 코틀린
---

## 가변성을 제한하라

## 들어가기에 앞서..

- 코틀린은 모듈로 프로그램을 설계한다.
- 모듈은 클래스, 객체, 함수, 타입별칭(`type alias`), 톱레벨 프로퍼티 등의 다양한 요소로 구성된다.
    - 요소 중 일부는 상태(var, mutable)를 가질 수 있다.
    
    ```kotlin
    var age = 20
    var bookList: MutableList<Book> = mutableListOf()
    ```
    
- 요소들이 상태를 갖는 경우 그 이력에도 의존하게 된다.(변화하는 값들의 이력)

## 요소들이 상태를 갖는 경우 그 이력에도 의존하게 된다.

```kotlin
class BankAccount {
		var balance = 0.0
				private set

		fun deposit(depositAmount: Double) {
				balance += depositAmout
		}
		
		@Throws(InsufficientFunds:class)
		fun withdraw(withdrawAmount: Double) {
				if (balance < withdrawAmount) {
						throw InsufficientFunds()
				}
				balance -= withdrawAmount
		}
}

class InsufficientFunds: Exception()
val account = BankAccount()
println(account.balance) // 0.0
account.deposit(100.0)
println(account.balance) // 100.0
account.withdrawAmount(50.0) // 50.0
```

위와 같은 상태를 갖는 경우

1. 프로그램을 이해하고 디버그하기 힘들어진다.
    - 상태 변경이 많아지면 이를 추적, 이해, 수정하기가 힘들다.
2. 가변성(mutablility)이 있는경우, 코드의 실행을 예측하기 어려워진다.
    - 시점에 따라 값이 변경될 수 있으므로, 현재 어떤 값을 가진지 정확히 알아야 실행을 예측할 수 있으며, 해당 값이 동일하게 유지된다고 확신할 수 없다.
3. 멀티스레드 프로그램일 경우 적절한 동기화가 필요하다.
    - 동시에 변경이 일어나는 부분에서 충돌이 발생할 수 있다.
4. 테스트하기 어렵다.
    - 모든 상태를 테스트해야 하기에, 변경이 많은경우 많은 조합을 테스트해야한다.
5. 상태 변경이 일어날 때, 추가적인 작업이 필요한 경우가 있다.
    - 정렬된 리스트에 가변 요소를 추가한 경우 요소에 변경이 일어날 때마다 리스트 전체를 다시 정렬해야한다.

## 공유 상태관리의 어려움

```kotlin
var num = 0
for (i in 1..1000) {
		thread {
				Thread.sleep(10)
				num += 1
		}
}
Thread.sleep(5000)
print(num) // 1000이 아닐 확률이 높으며 실행할 때마다 다른 숫자가 나온다.
```

- 위와 같이 멀티스레드를 활용해서 프로퍼티를 수정하는 경우, 충돌에 의해 일부 연산이 이루어지지 않는다.
- 일부 연산이 충돌되어 사라지므로 적절하게 동기화를 구현해야한다.
- 변화할 수 있는 지점이 많다면 더 어려워지므로, 변할 수 있는 지점은 줄일수록 좋다.

## 코틀린에서 가변성 제한하기

- 코틀린은 가변성을 제한할 수 있도록 설계되어 있다.
    - 불변(immutable)객체를 만들거나 프로퍼티를 변경할 수 없도록
- 많이 사용되고 중요한 것들로는
    1. 읽기 전용 프로퍼티 - val
    2. 가변 컬렉션과 읽기 전용 컬렉션 - List, MutableList 등
    3. 데이터 클래스의 copy

### 읽기 전용 프로퍼티(val)

```kotlin
val a = 10
a = 20 // 오류
```

- 코틀린은 val을 이용해 읽기 전용 프로퍼티를 만들 수 있다.
- 값(value)처럼 동작하며, 일반적인 방법으론느 값이 변하지 않는다.

```kotlin
val list = mutableListOf(1,2,3)
list.add(4)
print(list) // [1, 2, 3, 4]
```

- **읽기 전용 프로퍼티라고 해도 완전히 변경 불가능한 것은 아니다.**
- val을 사용해도 mutable 객체를 담고 있다면, 내부적으로 변할 수 있다.
    - 읽기 전용 프로퍼티는 재할당이 불가능한 것(프로퍼티를 읽을 수 있다)으로, 값이 변할 수 없는것(가변성)을 구분해야한다.

```kotlin
var name = "Marcin"
var surName = "Moskala"
val fullName: String
		get() = "$name $surname"

fun main() {
		println(fullName) // Marcin Moskala
		name = "AA"
		println(fullName) // AA Moskala
}
```

- 읽기 전용 프로퍼티는 다른 프로퍼티를 활용하는 사용자 정의 게터로도 정의가 가능하다.
- var 프로퍼티를 사용하는 val 프로퍼티는 var 프로퍼티가 변할 때 변할 수 있다.
- 값을 사용할 때마다 사용자 정의 게터가 호출 되므로 이러한 코드를 사용할 수 있다.

```kotlin
val name: String? = "철수"
val surname: String = "김"

val fullName: String?
		get() = name?.let { "$surname$name" }

val fullName2: String? = name?.let { "$surname$name" }

fun main() {
		if (fullName != null) {
				println(fullName.length) // 오류
		}
		if (fullName2 != null) {
				println(fullName2.length) // 3
		}
}
```

- **읽기 전용 프로퍼티지만 변경할수 없음(불변)을 의미하는 것은 아니다.**
- `fullName` 은 커스텀 게터로 정의했으므로 스마트 캐스트할 수 없다.
    - 값을 사용하는 시점의 name의 값에 따라 다른 결과가 나올 수 있기에
- `fullName2` 처럼 지역 변수가 아닌 프로퍼티가 final이고 커스텀 게터를 사용하지 안흔ㄴ 경우 스마트 캐스트할 수 있다.

### 가변 컬렉션과 불변 컬렉션 구분하기

- 코틀린에서는 읽기 전용 프로퍼티(val)와 읽고 쓸 수 있는 프로퍼티(var)를 구분해서 사용하며 이와 마찬가지로, **읽기 전용 컬렉션과 읽고 쓸 수 있는 컬렉션을 구분한다.**
- 컬렉션 계층이 설계된 방식에 따라 가변과 불변을 구분한다.

![스크린샷 2023-05-26 13.04.31.png](/assets/images/post/2023-05-26-collection.png)

```kotlin
val list = listOf(1,2,3)

// don't do this
if (list is MutableList) {
		list.add(4)
}
```

- 컬렉션 다운캐스팅은 추상화를 무시하는 행위이다.
- 위 코드의 실행 결과는 플랫폼에 따라 다르다.
    - JVM에서 `listOf`는 `List` 인터페이스를 구현한 `Array.ArrayList` 인스턴스를 리턴한다.
    - 자바의 `List` 인터페이스는 `add` 와 `set` 같은 메서드를 제공한다.
    - 코틀린에서는 MutableList로 변경이 가능하지만 자바에서는 `java.lang.UnsupportedOperationException` 이 발생한다.
- 읽기 전용에서 mutable로 변경해야 하는 경우 `copy` 를 통해 새로운 mutable 컬렉션을 만드는 `list.toMutableList` 를 활용해야 한다.
    
    ```kotlin
    val list = listOf(1,2,3)
    val mutableList = list.toMutableList()
    mutableList.add(4)
    ```
    

### 불변 객체를 많이 사용하는 이유

1. 한 번 정의된 상태가 유지되므로, 코드를 이해하기 쉽다.
2. 불변객체는 공유했을 때도 충돌이 이루어지지 않아서 안전하게 병렬처리를 할 수 있다.
3. 불변객체에 대한 참조는 변경되지 않으므로, 쉽게 캐시할 수 있다.
4. 불변객체는 방어적 복사본(defensive copy)을 만들 필요가 없다.
    - 따로 깊은 복사를 하지 않아도 된다.
5. 불변객체는 다른 객체를 만들 때 활용하기 좋다.
6. 불변객체는 `Set` 또는 `Map`의 키로 사용할 수 있다.
    - Set과 Map이 내부적으로 해시 테이블을 사용하고, 해시 테이블은 처음 요소를 넣을 때 요소의 값을 기반으로 버킷을 결정하기 때문
    - 요소에 수정이 일어나면 해시 테이블 내부에서 요소를 찾을 수 없다.

### 불변객체는 자신의 일부를 수정한 새로운 객체를 만들어 내는 메서드를 가져야한다.

- Int는 immutable이지만 내부적으로 `plus`와 `minus` 메서드를 통해 자신을 수정한 새로운 Int를 리턴할 수 있다.
- Iterable도 immutable이지만 `map` 과 `filter` 메서드를  통해 새로운 Iterable 객체를 만들어 리턴한다.
- **개발자가 만드는 불변객체도 비슷한 형태로 작동해야한다.**
    - 예를 들어 User라는 불변객체에서 성(surname)을 변경해야 한다면 그에 따른 메서드를 제공해서 자신을 수정한 새로운 객체를 만들어 낼 수 있게 해야한다.
    
    ```kotlin
    class User(
    		val name: String,
    		val surname: String,
    ) {
    		fun sithSurname(surname: String) = User(name, surname)
    }
    ```
    

### 데이터 클래스의 copy

- data 한정자를 사용한다면 `copy` 라는 이름의 메서드를 만들어 준다.
- `copy` 메서드를 활용하면, 모든 기본 생성자 프로퍼티가 같은 새로운 객체를 만들어 낼 수 있다.

```kotlin
data class User(
		val name: String,
		val surname: String,
)

var user = User("철수","김")
user = user.copy(surname = "이")
println(user) // User(name=철수, surname=이)
```

- 코틀린은 이와 같은 형태로 불변성을 가지는 데이터 모델 클래스를 만든다.

## 다른 종류의 변경 가능 지점

```kotlin
val list1: MutableList<Int> = mutableListOf() // val + mutable
var list2: List<Int> = listOf() // var + immutable

list1 += 1 // List1.plusAssign(1)로 변경됨.
list2 += 1 // list2 = list2.plus(1)로 변경됨.
```

- 위 코드는 두 가지 모두 += 연산자를 활용하지만 실질적으로 이루어지는 처리는 다르다.
- 두 방식의 변경 가능 지점의 위치가 다르다.
- val + mutable을 사용하는 경우 구체적인 리스트 구현 내부에 변경 가능 지점이 있다. 멀티스레드 처리가 이루어질 경우, 내부적으로 동기화가 되어 있는지 확실하게 할 수 없어 위험하다.
- var + immutable을 사용하는 경우 프로퍼티 자체가 변경 가능 지점이다. 따라서 멀티스레드 처리의 안정성이 더 좋다고 할 수 있다.

### 최악의 방식

```kotlin
var list3 = mutableListOf<Int>()
```

- 프로퍼티와 컬렉션 모두 변경 가능한 지점으로 만드는 것은 변경될 수 있는 두 지점 모두에 대한 동기화를 구현해야하며 모호성이 발생해서 +=를 사용할 수 없다.
- 상태를 변경할 수 있는 불필요한 방법은 만들지 않아야한다.
- 상태를 변경하는 모든 방법은 코드를 이해하고 유지해야 하기에 비용이 발생한다. 따라서 가변성을 제한하는 것이 좋다.

## 변경 가능 지점 노출하지 말기

- 상태를 나타내는 mutable 객체를 외부에 노출하는 것은 위험하다.
    - 돌발적인 수정이 일어날 때 위험할 수 있다.

### mutalbe객체 복제하기

- 리턴되는 mutable 객체를 복제하는 방어적 복제(defensive copy) 이용하기.
    - data class의 copy를 이용하면 좋다.

```kotlin
class UserHolder {
		private val user: MutableUser()
		fun get(): MutableUser {
				return user.copy()
		}
}
```

## 정리

- 가능한 경우 무조건 가변성을 제한하는 것이 좋다.
- var 보다는 val을 사용하는 것이 좋다.
- mutable 프로퍼티보다는 immutable 프로퍼티를 사용하는 것이 좋다.
- mutable 객체와 클래스 보다는 immutable객체와 클래스를 사용하는 것이 좋다.
- 변경이 필요한 대상을 만들어야하는 경우, immutable 데이터 클래스로 만들고 copy를 활용하는 것이 좋다.
- 컬렉션에 상태를 저장해야 하는경우, mutable 컬렉션보다는 읽기 전용 컬렉션을 사용하는 것이 좋다.
- 변이 지점을 적절하게 설계하고, 불필요한 변이 지점은 만들지 않는 것이 좋다.
- mutable객체를 외부에 노출하지 않도록 한다.