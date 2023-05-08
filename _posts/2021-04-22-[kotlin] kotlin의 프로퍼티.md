---
layout: "post"
title: "[kotlin] kotlin의 프로퍼티"
subtitle: "kotlin property"
date:       2021-04-22
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
    - property
---


## 프로퍼티 (Property)

> 코틀린에서는 클래스내의 변수(var, val 로 선언되는)를 `프로퍼티(Property)`라 부릅니다. 그 이유는 흔히 `접근자라고 부르는 getter와 setter 함수가 내장`되어 있습니다.


### 프로퍼티를 작성하는 형식

```
[] 대괄호는 생략 가능을 의미

var <propertyName>[: <PropertyType>] [= <property_initializer>]
    [<getter>]
    [<setter>]
```

 - 프로퍼티는 var나 val로 선언하며 이름과 데이터 타입, 초깃값을 명시할 수 있습니다.
    
 - 프로퍼티 선언시 getter 와 setter는 생략할 수 있습니다.

    - `내부적으로 getter 와 setter 함수가 추가 되며`<br> 변경이 가능한 var 로 선언시 `getter 와 setter` 가 추가 되고<br> 변경이 불가능한 val 로 선언시 `getter 만` 추가 됩니다.

 - 데이터 `타입이 추론할 수 있는 상황이라면 생략`할 수 있습니다.

ex)
```kotlin
class User{
    var name: String = "Lee"
        get() = "name is $field"  // field 는 해당 프로퍼티의 실제  값이다
        set(value) { field = value }

    val age: Int = 20
        get() = field 
}
// ...

val user = User()
user.name = "Kim"       // name프로퍼티의 set(value) 호출
println(user.name)      // name프로퍼티의 get() 호출
```

 - get(), set() 함수의 `field는 프로퍼티에 저장된 값 자체를 지칭`하는 예약어로 get(),set() 함수에서만 사용할 수 있습니다.
  
 - 외부에서 프로퍼티를 이용할때는 get(), set()함수가 호출되고, 해당 메서드 내부에서는 field 예약어를 통해 프로퍼티가 가지는 값에 접근 합니다. 이를 `배킹필드(BackingField)` 라고 부릅니다.



### 초기화 블록에서 초기화

 - 프로퍼티를 선언하고 초기화 블록(init) 에서 프로퍼티를 초기화해 사용할 수 있습니다.

ex)
```kotlin
class User{
    var name : String
    val age : Int

    init{
        name = "Lee"
        age = 27
    }
}
```

### 늦은 초기화 (lateinit)

 - 프로퍼티의 타입을 nullable하게 한뒤 초깃값으로 null을 주고 이후 값을 대입해 사용하는 방법도 있지만 nonNull 타입의 프로퍼티의 초기화를 미루는 방법도 있습니다. 이 방법은 `lateinit` 예약어를 이용하며 `늦은 초기화` 라고 합니다.

ex)
```kotlin
class User{
    lateinit var lateName : String
}
// ...

val user = User()
user.lateName = "Lee"
```

 - `lateinit(늦은 초기화) 사용시 주의사항`

    - var로 선언한 프로퍼티에서만 사용이 가능 합니다.

    - 클래스 내부에 선언한 프로퍼티만 사용할 수 있다. 주 생성자에서 사용 불가

    - 사용자 정의 getter/setter 를 사용하지 않은 프로퍼티에만 사용할 수 있습니다.

    - nullable 한 타입의 프로퍼티는 사용할 수 없습니다.

    - 기초 타입 프로퍼티에는 사용할 수 없습니다.

### 초기화 미루기 (by lazy)

 - 프로퍼티의 초기화를 프로퍼티의 `이용 시점 으로 미루는 방법` 입니다.

 - `호출 시점에 초기화가 진행 됩니다.`
 
 - val 로 선언한 프로퍼티에서만 사용이 가능 합니다. 

 - 클래스 몸체 이외에 최상위 레벨에서도 사용 가능 합니다

 - 기초 타입에도 사용이 가능 합니다.
 
 ex)
```kotlin
val lazyData : String by lazy{
    println("lazy...")  // 초기화 당시 출력
    "it's LazyData"     // 해당 프로퍼티의 값
}

fun main(){
    println("main start")
    println(lazyData) // lazyData 호출시 위의 lazy 구문 실행, 마지막 줄 데이터로 초기화 됨(it's LazyData).
}
```

결과
```
main start
lazy...
it's LazyData
```


  


---

참조 : 깡샘의 코틀린 프로그래밍
