---
layout: "post"
title: "[TIL] Kotlin - 제네릭 타입과 variance 한정자를 활용하라."
subtitle: "Effective kotlin - Item 24"
date: 2023-05-23
author: "raindragonn"
lang: ko
categories: [TIL - Kotlin]
tags:
    - Kotlin
    - Effective Kotlin
    - 이펙티브 코틀린
---

# 제네릭 타입과 variance 한정자를 활용하라.

## 코틀린의 타입파라미터

<aside>
💡 코틀린의 제네릭은 기본적으로는 불공성이다. (한정자 가 없는 경우)

</aside>

### 불공변성, 무공변성 (Invariant)

- 제네릭 타입으로 만들어지는 타입들이 서로 관련성이 없다를 의미
- `Exam<Int>` 와 `Exam<Number>` 은 어떠한 관련성도 갖지 않는다.

```kotlin
class Exam<T>

fun main() {
		val anys: Exam<Any> = Exam<Int>() // Error: Type missmatch
		val nothings: Exam<Nothing> = Exam<Int>() // Error
}
```

## variance 한정자

<aside>
💡 제네릭 타입으로 만들어지는 타입들이 서로 어떤 관련성을 같기를 원하는 경우 사용.

</aside>

### out - 공변성 (Covariant)

- 한정자로 `out` 을 사용한다.
    - **out 키워드가 붙은 타입 파라미터를 공변성으로 만든다.**
- A가 B의 서브 타입인 경우 Exam<A>가 Exam<B>의 서브타입을 의미함.
- **자기 자신과 자식 타입을 허용한다.**

```kotlin
class OutExam<out T>
open class Dog
class Puppy: Dog()

fun main(args: Array<String>) {
		// OK - Puppy는 Dog의 서브클래스
		val b: OutExam<Dog> = OutExam<Puppy>()

		// Error - Dog는 Puppy의 서브클래스가 아님
		val a: OutExam<Puppy> = OutExam<Dog>()
		
		// OK - Int는 Any의 서브클래스
		val anys: OutExam<Any> = OutExam<Int>()

		// Error Int는 Nothing의 서브클래스가 아님
		val nothings: OutExam<Nothing> = OutExam<Int>()
}
```

### in - 반공변성, 반변성 (Contravariant)

- 한정자로 `in` 을 사용한다.
    - **in 키원드가 붙은 타입 파라미터를 반공변성으로 만든다**
- A가 B의 서브타입일 때, Exam<A> 가 Exam<B>의 슈퍼타입이라는 것을 의미합니다.
- **자기 자신과 부모타입을 허용한다.**

```kotlin
class InExam<in T>
open class Dog
class Puppy: Dog()

fun main(args: Array<String>) {
		// Error - Puppy는 Dog의 슈퍼클래스가 아님
		val b: InExam<Dog> = InExam<Puppy>()

		// OK
		val a: InExam<Puppy> = InExam<Dog>()
	
		// Error - Int는 Any의 슈퍼클래스가 아님
		val anys: InExam<Any> = InExam<Int>()

		//OK
		val nothings: InExam<Nothing> = InExam<Int>()
}
```

## 함수 타입

- 아래에서 파라미터 `task` 의 타입

```kotlin
fun exam(task: (Int) -> Any) {
		print(task(414))
}
```

- 함수 타입은 파라미터 유형과 리턴 타입에 따라 서로 어떤 관계를 갖는다.
- 위의 task의 타입은 `(Int) → Number`, `(Number) → Any` , `(Number) → Number`, `(Number)→Int` 등으로도 동작됨.
    
    ![스크린샷 2023-05-17 오후 3.14.17.png](/assets/images/post/2023-05-23-type.png)
    
    - 이러한 동작이 가능 한 이유는 아래와 같은 관계가 있기 때문
    
    ![스크린샷 2023-05-17 오후 3.14.54.png](/assets/images/post/2023-05-23-hierarchy.png)
    
- 파라미터 타입이 더 높은 타입으로 이동하고, 리턴 타입은 계층 구조의 더 낮은 타입으로 이동
- 함수 타입의 모든 **파라미터 타입은 Contravariant 즉, 반공변성을 가진다. (in 키워드)**
- 함수 타입의 모든 **리턴 타입은 Covariant 즉, 공변성을 가진다. (out 키워드)**

## variance 한정자의 안전성 - (자바와의 차이점)

- 자바에서의 배열은 covariant이다.
- 그에 따른 문제로 컴파일 중에는 문제가 없지만 런타임 오류가 발생하는 경우가 있습니다.

```java

Integer[] numbers = {1, 4, 2, 1};
Object[] objects = numbers;
objects[2] = "B"; // 런타임 오류: ArrayStoreException
```

- numbers를 Object[]로 캐스팅 해도 내부의 실질적인 타입이 바뀌는 것이 아님
- 코틀린은 이러한 결함을 해결하기 위해서 Array를 invariant로 만들습니다.
- ex) IntArray, CharArray

## 코틀린에서 허용하지 않는 상황

### 코틀린의 특징

1. 파라미터 타입을 예측할 수 있다면, 어떤 서브타입이라도 사용가능하다.
2. 암묵적으로 상위 타입으로 업캐스팅할 수 있다.

```kotlin
open class Dog
class Puppy: Dog()
class Hound: Dog()

fun takeDog(dog: Dog) {}

takeDog(Dog())
takeDog(Puppy())
takeDog(Hound())
```

```kotlin
open class Car
interface Boat
class Amphibious: Car(), Boat
fun getAmphibious(): Amphibious = Amphibious()
val car: Car = getAmphibious() 
val boat: Boat = getAmphibious()
```

### 불허 1.public in 위치와 out 한정자 문제

<aside>
🚫 covariant 타입 파라미터(out)가 in 한정자 위치(함수의 타입 파라미터)에 있다면, covariant(out)와 업캐스팅을 연결해서 우리가 원하는 타입을 아무것이나 전달할 수 있는 문제가 생길 수 있음.

</aside>

```kotlin
class Box<out T> {
    // Type parameter T is declared as 'out' but occurs in 'in' position in type T
    var value: T? = null

    // Type parameter T is declared as 'out' but occurs in 'in' position in type T
    fun set(value: T) {
        this.value = value
    }

    fun get(): T = value ?: error("Value not set")
}

fun main() {
    val puppyBox = Box<Puppy>()
    val dogBox: Box<Dog> = puppyBox

    //실제 값은 Box<Puppy> 였지만 업캐스팅으로 인해 Box<Dog>라서 Hound를 사용 가능한 문제가 생김.
    dogBox.set(Hound())
}
```

- 위 코드는 실행 불가.
- public in 한정자 위치에 convariant 타입 파라미터(out)가 오는 것을 금지한다.

### 허용

- 객체 내부에서는 업 캐스트 객체에 covariant(out 한정자)를 사용할 수 없기 때문에, 가시성을 private로 제한한 경우에 오류가 발생하지 않는다.

```kotlin
class Box<out T> {
		private var value: T? = null 

		private fun set(value: T) {
				this.value = value
		}
		fun get(): T = value ?: error("Value not set")
}
```

- out 한정자는 public out 한정자 위치에서도 안전하므로 따로 제한 되지 않는다.
- 이러한 안정성의 이유로 생성되거나 노출되는 타입에만 out 한정자를 사용한다.
- 일반적으로 producer 또는 immutable 데이터 홀더에 많이 사용된다.

### 불허 2.public out 위치와 in한정자 문제

```kotlin
class Box<in T> {
		// 코틀린에서는 사용할 수 없는 코드
		val value: T
}

val garage: Box<Car> = Box(Car())
val amphibiousSpot: Box<Amphibious> = garage
//garage의 인스턴스는 Box<Car> 이지만 Box<Amphibious>로 캐스팅할 수 있는 문제
val boat: Boat = garage.value
```

- 위 코드는 실행 불가.
- public out 한정자 위치에 contravariant 타입 파라미터(in)가 오는 것을 금지한다.

### 허용

```kotlin
class Box<in T> {
		private var value: T? = null
		
		fun set(value: T) {
				this.value = value
		}

		private fun get(): T = value
				?: error("Value not set")
}
```

- in 한정자는 public in 한정자 위치에서 안전하므로 따로 제한 되지 않는다.
- 이러한 이유로 타입 파라미터에 in 한정자를 사용한다.

### 좋은 예시 1. List와 MutableList

- 좋은 예로 T는 covariant(out)인 List<T>가 있다.
    - `public interface List<out E> : Collection<E>`
- 함수의 파라미터가 List<Any?>로 예측된다면, 별도의 변환 없이 모든 종류를 파라미터로 전달할 수 있다.
- MutableList<T>에서 T는 in 한정자 위치에서 사용되어 안전하지 않으므로 invariant이다.
    - `public interface MutableList<E> : List<E>, MutableCollection<E>`

```kotlin
fun append(list: MutableList<Any>) {
		list.add(42)
}

val strs = mutableListOf<String>("A","B","C")
append(strs) // Error
val str: String = strs[3]
print(str)
```

### 좋은 예시 2. Response

```kotlin
sealed class Response<out R, out E>
class Failure<out E>(val error: E): Response<Nothing, E>()
class Success<out R>(val value: R): Response<R, Nothing>()
```

- 아래 내용은 모두 참이다.
    1. Response<T>라면 T의 모든 서브타입이 허용된다.
        - Response<Any>가 예상된다면, Response<Int>, Response<String>도 허용
    2. Response<T1, T2> 라면 T1과 T2의 모든 서브타입이 허용된다.
    3. Failure<T>라면 T의 모든 서브타입이 허용된다.
        - 1번과 동일
    4. covariant(out 한정자)와 Nothing 타입을 활용하면 Failure와 Success에서 각 값과 오류 타입을 지정하지 않아도 된다. -**확인 필요 143페이지**

## variance 한정자의 위치

### 1. 선언 부분에서의 사용 - 클래스 및 인터페이스 선언

- 클래스와 인터페이스가 사용되는 모든 곳에 영향을 준다.

```kotlin
// 선언 쪽의 variance 한정자
class Box<out T>(val value: T)
val boxStr: Box<String> = Box("str")
val boxAny: Box<Any> = boxStr
```

### 2. 클래스와 인터페이스를 활용하는 위치에서 사용

- 한정자를 사용하면 특정한 변수에만 variance 한정자가 적용된다.

```kotlin
class Box<T>(val value: T)
val boxStr: Box<String> = Box("Str")
// 사용하는 쪽의 variance 한정자
val boxAny: Box<out Any> = boxStr
```

- 모든 인스턴스가 아닌 특정 인스턴스에만 적용해야할 때 사용.

### 위치 제한

- variance한정자를 사용하는 경우 위치가 제한될 수 있다.
- MutableList<out T> 에서 get은 T타입이 나올 것이지만 set은 Nothing 타입이 가능하기에 사용할 수 없다.
    - 모든 타입의 서브타입을 가진 리스트(Nothing 리스트)가 존재할 가능성이 있기에
- MutableList<in T>를 사용할 경우, get과 set 모두 사용할 수 있지만, get을 사용할 경우 자료형은 Any?가 된다.
    - 모든 타입의 슈퍼타입을 가진 리스트(Any 리스트)가 존재할 가능성이 있기에
    

## 정리

- 코틀린은 variance 한정자를 통해 타입 아규먼트의 관계에 제약을 걸 수 있는 제네릭 기능을 제공한다.
1. 타입 파라미터의 기본은 invariant다.
    - A가 B의 서브타입이라고 할 때에도 Cup<A>와 Cup<B>는 아무런 관계를 갖지 않는다.
2. out 한정자는 타입 파라미터를 covariant하게 만든다. (공변)
    - A가 B의 서브타입이라고 할 때 Cup<A>는 Cup<B>의 서브타입이 된다. (Cup<out T>)
3. in 한정자는 타입 파라미터를 contravariant하게 만든다. (반공변 
    - A가 B의 서브타입이라고 할 때 Cup<B>는 Cup<A>의 슈퍼타입이 된다. (Cup<in T>)
4. List와 Set그리고Map의 값은 타입 파라미터는 out 한정자를 사용한다.
    - List<Any>가 예상되는 모든 곳에 전달할 수 있다.
5. Array, MutableList, MutableSet, MutableMap의 타입 파라미터는 invariant(한정자 없음 - 불공변성)
6. 함수 타입의 파라미터 타입은 contravariant(in 한정자)이며, 리턴 타입은 covariant(out 한정자)
7. 리턴만 되는 타입에는 covariant(out 한정자)를 사용한다.
8. 허용만 되는 타입에는 contravariant(in 한정자)를 사용한다.