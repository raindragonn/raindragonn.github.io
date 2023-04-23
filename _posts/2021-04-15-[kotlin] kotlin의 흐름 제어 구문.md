---
layout: "post"
title: "[kotlin] kotlin의 흐름 제어 구문"
subtitle: "if, when, for, while"
date:       2021-04-15
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
    - if, when, for, while
---

## 조건문

### if 표현식
> `조건에 맞으면 특정 영역을 실행하는 구문입니다.`<br>  kotlin 에서의 특징으로는 `if문이 표현식(expression)` 이라는 점입니다.


**일반적인 if문 ex)**
```kotlin
val max = 5

fun isMax(value : Int) : Boolean {
    return if(max <= value){
        true
    }else{
        false 
    }
}
```

`if을 표현식으로 사용할 때 주의해야 할 점은 else 문이 꼭 있어야 한다는 점입니다.`

**표현식 ex)**
```kotlin
val result = if(max > 10) "크다" else "작다"
```

### when 표현식

> `인수로 지정한 값과 일치하는 분기 조건을 찾아서 실행`하는 switch 문을 대체합니다.<br>
> kotlin 에서의 특징 으로는 정수 이외에도 다양한 타입의 데이터를 지정, 아예 타입 자체를 지정할 수도 있습니다. `if문과 동일하게 표현식` 입니다.


**정수 분기 ex)**
```kotlin
val max = 0 
when(max){
    0 -> println("max는 0") 
    1 -> println("max는 1")
    else -> println("max는 0도 1도 아님")
}
```

`표현식으로 사용하지 않을때는 else문을 생략할수 있다.`

**문자열 분기, else 생략 ex)**
```kotlin
val data = "hello"
when(data){
    "hello" -> println("data = hello")
    "world" -> println("data = world")
}
```

`조건식은 콤마 , 로 묶을수 있다.`

**여러 조건 분기 ex)**
```kotlin
val data = 1
when(data){
    0,1 -> println("data = 0 or 1")
    else -> println("data != 0 or 1")
}
```

`범위 연산자 in 을 이용할 수 있다.`

**범위 연산자 사용 ex)**
```kotlin
val data = 10
val listData = listOf(10,20,30)
when(data){
    in 1..100 -> println("data in 1..100")
    in listData -> println("data in listData")
}
```

`여러 타입으로도 분기가 가능하다.`

**ex)**
```kotlin
fun isHelloStart(data: Any) = when(data) {
    is String -> x.startsWith("hello")
    else -> false
}
```

`when에 인수를 전달하지 않고 if-else 대체용으로 사용이 가능하다.`

**ex)**
```kotlin
val max = 10
when{
    max < 10 -> println("max는 10보다 작다")
    max > 10 -> println("max는 10보다 크다")
    else -> println("max는 10이다")
}
```

`if와 마찬가지로 표현식으로 사용할수 있으며, else문이 있어야 합니다.`

**ex)**
```kotlin
val result = when{
    max > 10 -> "크다"
    max < 10 -> "작다"
    else -> "같다"
}
```

## 반복문

### for 반복문

> `for 안에 변수를 선언하고 특정 조건에 맞을 때까지 구문을 반복 실핼하는 구문입니다.`

**ex)**
```kotlin
var sum = 0
// 0 부터 10까지 반복
for(i in 0..10){
    sum += i
}

//0 부터 9 까지 반복(10을 포함하지 않음)
for(i in 0 until 10){
    sum += i
}

// 0부터 10까지 2씩 증가 하면서 반복
for(i in 0..10 step 2){
    sum += i 
}

// 10부터 1까지 반복
for(i in 10 downTo 1){
    sum += i
}
```

`컬렉션 타입의 데이터를 데이터 개수만큼 순차 반복할 수 있습니다.`

**ex)**
```kotlin
var sum = 0
val listValue : List<Int> = listOf(1,2,3)

for(i in listValue){
    sum += i
}
```

`컬렉션 타입 데이터의 인덱스 값을 이용할땐 컬렉션의 indices 프로퍼티를 이용하면 됩니다.`

**ex)**
```kotlin
var sum = 0
val listValue : List<Int> = listOf(1,2,3)

for(i in listValue.indices){
    sum += i
}
```

`컬렉션 타입의 데이터와 인덱스값 모두를 얻고 싶을땐 withIndex() 를 이용 합니다.`

**ex)**
```kotlin
val listValue : List<Int> = listOf(1,2,3)
for((index,value) in listValue.withIndex()){
    println("index = $index, value = $value")
}
```

### while 반복문

> while에 명시한 조건이 맞으면 {} 부분을 반복 수행합니다.

**ex)**

```kotlin
var x = 0
var y = 0

while(x < 10){
    println("${++x}번 반복중")
}

do{
    println("${++y}번 반복중")
} while(y < 10)
```

### break, continue

> kotlin에서도 while,for 등의 반복문에서 break, continue를 이용하여 <br>
> 반복문을 넘어가거나 나갈수 있습니다.

```kotlin
var y = 0

do {
    if (++y % 2 != 0) {
        continue
    } else {
        if (y > 10) break
        println("짝수 = $y")
    }
} while (true)
```

---

참조 : 깡샘의 코틀린 프로그래밍
