---
layout: "post"
title: "[kotlin] kotlin 의 데이터 타입"
subtitle: "Data type"
date:       2021-04-14
author: "raindragonn"
banner:
  image: "/assets/images/base/banner.jpg"
  opacity: 0.618
  height: "38vh"
  min_height: "38vh"
  heading_style: "font-size: 2.25em; font-weight: bold; text-decoration: underline"
  subheading_style: "color: gold"
lang: ko
categories: [TIL - Kotlin]
tags:
    - kotlin basic
    - DataType
---

# 데이터 타입

코틀린에서 모든 것은 `객체(Object)`입니다.

데이터 타입을 명시할 때 다른 언어(C나 자바)에서는 int, double 등의 기초 타입과 Integer,Double 등의 Wrapper 클래스로 구분해서 사용하지만, 코틀린에서는 `기초 타입이 없습니다`.

***ex)***

```kotlin
val intValue : Int = 10
val result = intValue.minus(5)  // int형 객체 intValue 에서 minus 메서드 호출
```


## 숫자 타입 

> 코틀린의 숫자 타입 클래스들은 모두 Number 클래스의 서브 클래스 입니다.




`숫자 타입별 크기`

|Type|Bit Width|
|:---:|:---:|
|int|32|
|Short|16|
|Byte|8|
|Float|32|
|Double|64|
|Long|64|

<br>

`숫자 타입에 값을 대입할 때는 다음과 같은 규칙이 있다.`

- Long형은 값 뒤에 'L' 사용
- 실수 기본형은 Double
- 실수형은 10.0e2 로도 표현 가능
- Float 형은 값 뒤에 'f','F' 사용
- 숫자를 조금 더 읽기 좋게 표한하기 위한 표현식으로 밑줄 _ 를 추가할수 있다.

ex)

```kotlin
val byteValue : Byte = 0b00001011
val intValue : Int = 123
val intValue2 : Int = 0x0F
val longValue : Long = 10L
val doubleValue : Double = 10.0
val doubleValue2 : Double = 123.5e10
val floatValue : Float = 10.0f
val underScoreValue : Int = 1_000_000
```


## 논리, 문자와 문자열 타입

> Boolean 타입은 true, false값을 표현하기 위한 타입입니다.

다음의 연산자도 제공된다

- `||`-> 논리합
- `&&` -> 논리곱
- `!` -> 부정

**ex)**
```kotlin
val isTrue1 : Boolean = true || false // 둘중 하나라도 true 면 true
val isTrue2 : Boolean = true && false // 둘다 true 여야 true
val isTrue3 : Boolean = !true // true를 false 로, false를 true로
```

Char는 문자를표현하는 타입이며 ''(작은 따옴표)로 묶어서 표현합니다.

String 은 문자열을 표현하는 타입이며 Char의 집합입니다.

**ex)**
```kotlin
val charValue : Char = 'c'
val stringValue : String = "string"
```

### 문자열 템플릿 

> 문자열 내에 변수의 데이터나 특정 연산식 결과에 의한 데이터를 `$기호`로 쉽게 포함할수 있다.

**ex)**
```kotlin
val name : String = "lee"
println("이름은 ${name}")
```

## Any 타입

> 코틀린 클래스의 최상위 클래스 입니다.

타입을 선언할 때 Any를 이용하는 것은 어떤 타입의 데이터도 대입할 수 있다는 것이며,

특정 변수에 대입되는 타입을 예측할 수 없을 때 유용하게 사용할 수 있습니다.

***ex)***

```kotlin
fun getLength(obj : Any) : Int{
    if(obj is String){
        return obj.length
    }
    return 0
}

fun main(){
    println(getLength("hi"))
    println(getLength(2))
}
```

***결과***

```
2
0
```

## null 허용 타입

> 코틀린에서는 null을 대입하려면 명시적으로 nullable 한 타입임을 ? 기호를 이용하여 표현하여야 합니다.

**ex)**
```kotlin
val a : Int = null  // Null can not be a value of a non-null type Int
val b : Int? = null 
```

## 컬렉션 타입

### 배열 (Array)

> 코틀린에서는 배열을 array로 표현된다. get(), set(), size() 등의 함수를 표현하는 클래스입니다.


가장 쉽게 배열을 만드는 방법은 `arrayOf()`를 이용하면 됩니다.

`arrayOfNulls<T>(size : Int)`를 이용하여 초깃값이 비어있는 배열을 만들수 있습니다.

***ex)***
```kotlin
val arr = arrayOf("hi",1,true)
val arr2 = arrayOf<Int>(1,2,3) //제네릭을 통한 타입 한정
val arr3 = Array(3){i ->    
    i * 10
}
```

위의 예시 처럼 제네릭으로 배열을 선언하지 않아도 지정된 함수를 이용하여 특정 타입의 배열을 만들 수 있습니다.

- booleanArrayOf()
- byteArrayOf()
- charArrayOf()
- doubleArrayOf()
- floatArrayOf()
- intArrayOf()
- longArrayOf()
- shortArrayOf()

## List, Set, Map (Collection)

`Collection 종류`

- List : 순서가 있는 데이터 집합, 데이터의 중복 허용
- Set : 순서가 없으며 데이터 중복을 허용하지 않음
- Map : 키와 값으로 이루어지는 데이터 집합, 순서가 없으며 키의 중복을 허용하지 않음

> 코틀린에서 컬렉션 타입의 클래스들은 가변 클래스와 불변 클래스로 구분 됩니다. 해당 구분에 따라 가변의 경우 데이터 추가 및 변경하는 함수를 제공 하기도 합니다.


|분류|타입|함수|특징|
|---|---|---|---|
|`List`|List|listOf()| 불변
||MutableList|mutableListOf()|가변
|`Map`|Map|mapOf()|불변
||MutableMap|mutableMapOf()|가변
|`Set`|Set|setOf()|불변
||MutableSet|mutableSetOf()|가변


***ex)***
```kotlin
val list : List<String> = listOf("hello","world")
val mutableList : MutableList<String> = mutableListOf("hi","world")

val map : Map<String,Int> = mapOf(Pair("hi",1)) //Pair 클래스 이용
val mutableMap : Map<String,Int> = mapOf("hi" to 1)
val mutableMap2 : MutableMap<String,Int> = mutableMapOf("hi" to 1)

val set : Set<Int> = setOf(1,1,2)
val mutableSet : MutableSet<Int> = mutableSetOf(1,1,2)
```

### 이터레이터 iterator

> 이터레이터는 컬렉션 타입의 데이터를 차례로 얻어서 사용하기 위한 인터페이스 입니다.

해당 인터페이스에 내장된 메서드에는 두가지가 있습니다.

`public operator fun next(): T` - 실제 데이터를 가져오는 함수

`public operator fun hasNext(): Boolean` - 가져올 수 있는 데이터가 있으면 true, 없으면 false 반환

**ex)**

```kotlin
fun main(){
    val listValue : List<String> = listOf("hello","world")

    val iterator = listValue.iterator()

    while (iterator.hasNext()){
        println(iterator.next())
    }
}
```

**결과**
```
hello
world
```




---

참조 : 깡샘의 코틀린 프로그래밍