---
layout: "post"
title: "[kotlin] kotlin 의 변수 및 함수"
subtitle: "var, val, fun"
author: "raindragon"
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
---



## 코틀린이란?

 > `intelliJ로 유명한 젯브레인의 오픈소스 그룹에서 개발한 프로그래밍 언어이다.`

## 코틀린의 특징

1. `자바, 안드로이드 100% 호환성`

   - 코틀린으로 개발된 코드는 자바 클래스로 빌드되어 JVM 에서 동작 한다.<br>
   kt 파일은 자바의 class 파일로 바뀌어 실행

2. `null 안전성`

     - 기본적으로 데이터 타입에 nullable, non-nullable 구분을 확실시 한다.
  
3. `확장 함수`
   
   - 상속을 통하지 않고 Extension을 이용해 기존 클래스에 기능을 추가할 수 있다.

4. `함수형 프로그래밍`
   
   - 람다식과 고차함수를 지원 해서 함수형 프로그래밍이 가능하다.


## 변수 특징


### 변수 선언법

코틀린에서 변수는 `val`(value) 혹은 `var`(variable) 키워드를 이용하여 선언 합니다.<br>

두가지의 키워드는 `Assign-onece(Read-only)`와 `Mutable`로 구분됩니다.<br>

즉, `val` 은 선언 할때 값 이후 변경이 불가하며 `var` 는 지속적인 변경이 가능합니다.

또한 코틀린은 데이터 타입을 명시 하지 않아도 `타입 추론`을 통해 생략이 가능합니다.

형식

    var (혹은 val) 변수명 : 변수 타입 = 값

**ex)**
```kotlin
val name : Int = 1
name = 2    // Val cannot be reassigned Error
    
var name2 : Int = 2
name2 = 3   // 정상 동작

var name3 = 3 // 타입 추론을 통한 데이터 타입 생략
```

### null 값 적용

코틀린 에서는 `null 값을 대입하려면 명시적으로 null이 될수 있는 변수(nullable)로 선언`해야 합니다.  

이는 코틀린이 null에 안전한 프로그램을 작성하기위한 특징인 `null 안전성` 떄문입니다.  

null값을 허용하는 변수를 선언하기 위한 방법은 `?(물음표)` 기호를 이용해 명시적으로 null 이 될 수 있는 변수로 선언 해야합니다.

**ex)**
```kotlin
var nonNullData : String = null //Null can not be a value of a non-null type String

var nullableData : String? = null // 정상 동작
```

### 상수변수 선언

일반적으로 처음에 대입한 초깃값을 변경하지 않고 그대로 사용하는 변수를 `상수변수` 라고 한다.  

코틀린에서는 `val`로 변수 선언을 해도 초깃값과 달라질수 있습니다.  

그 이유는 코틀린에서 변수는 내부적으로 getter와 setter를 제공하는 `프로퍼티` 이기 때문입니다.  

정확히는 val 의경우 getter만, var의 경 getter와 setter 모두 제공 합니다.
따라서,`get()의 처리 내용에 따라 값이 달라질수 있습니다.`

**ex)**
```kotlin
var isWorld = false
    
fun String.addWorld() = "${this}World" // 코틀린의 확장함수를 이용

val testString : String = "Hello"
    get() {
        return if(isWorld.not())
            field       // 해당 프로퍼티의 backing Filed
            else
            field.addWorld()
    }
    
fun main(args : Array<String>){
    println(testString)
    isWorld = true
    println(testString)
}
```

**출력**
```
Hello
HelloWorld
```


일반적으로 말하는 `상수변수`(처음에 대입한 값을 변경할 수 없고 항상 초깃값만 반환하는)는 `const`라는 예약어를 사용해야 합니다.

`const` 예약어를 통해 변수를 선언하면 `런타임이 아닌 컴파일 시간에 값을 할당`합니다.

때문에 제약사항으로 클래스의 프로퍼티나 지역변수로 할당할수 없고, `primitive type(원시타입)과 String`만 넣을수 있습니다.


**ex)**
```kotlin
//제약사항으로 최상위 레벨에서 선언하거나 object 안에서 선언 해야한다.
const val constValue : String = "constValue"

companion object {
    const val constValue : String = "constValue"
}
```

## 함수 특징

### 함수 선언

**형식**
```
fun 함수명(매개변수명: 타입) : 반환타입{내용}
```

반환타입을 받고 싶지 않을때는 반환타입을 Unit 혹은, 생략 가능하다.

단일 구문으로 값을 반환하는 함수라면 `중괄호 {}를 생략 하고 등호 = 를 이용해 나타낼수 있다.`

 `기본인수` 즉, 매개변수를 대입하지 않을 때 자동으로 적용되는 기본값을 넣을수 있다.

**ex)**
```kotlin

fun sum1(a : Int, b: Int) : Int {
    return a + b
}

// 반환타입을 받고 싶지 않을때는 반환타입을 Unit 혹은, 생략 가능하다.
fun sum2(a: Int, b: Int): Unit {
    println(a + b)
}

// 단일구문
fun sum3(a: Int, b: Int): Int = a + b

// 반환타입 생략 가능
fun sum4(a: Int, b: Int) = a + b

// 기본인수 사용
fun sum5(a: Int = 10, b: Int = 20) = a + b
```

### 가변 인수

함수는 매개 변수를 다르게 하여 같은 이름의 함수를 여러 개 정의할 수 있습니다.

이를 함수 `오버로딩` 이라고 합니다.

하지만, 오버로딩에 의해 너무 많은 함수가 선언한다면 오히려 불편할 수 있습니다.

이때, `가변 인수` 를 사용하면 쉽게 작성 할 수 있습니다.

가변 인수를 사용하려면 인자 앞에 `vararg`를 붙이면 된다

**ex)**
```kotlin
// 가변 인수를 T 타입 으로 받아 List<T> 로 반환
fun <T> asList(vararg ts: T): List<T> {
    val result = ArrayList<T>()
    for (t in ts) // ts is an Array
        result.add(t)
    return result
}

val list = asList(1,2,3)
```

참조 : 깡샘의코틀린 프로그래밍