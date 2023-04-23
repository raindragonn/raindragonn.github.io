---
layout: "post"
title: "[kotlin] kotlin의 추상클래스, 인터페이스"
subtitle: "abstract class, interface"
date:       2021-04-24
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
    - abstract class
    - interface
---

## 추상 클래스 `abstract class`

> 추상 클래스란 `추상 함수를 포함하는 클래스`를 의미 합니다.


**추상함수, 추상 프로퍼티**

 - 추상 함수는 미완성 함수 혹은 `실행 영역이 없는 함수`를 의미합니다.
 
 - 추상 함수는 반드시 `함수 선언 부분에 abstract 예약어가` 있어야 합니다.

 - 추상 프로퍼티는 `초깃값 없이 선언만 있는 프로퍼티`를 말합니다.

 - 추상 프로퍼티는 반드시 `프로퍼티 선언 부분에 abstract 예약어가` 있어야 합니다.

 - 추상 함수, 추상 프로퍼티는 상속과 관련이 있어 `클래스 내부`에서만 사용할 수 있습니다.

### 추상 클래스 선언

 - 추상 함수를 포함하는 클래스는 꼭 `클래스 선언 부분에 abstract 예약어`가 있어야 합니다.

 - 추상 프로퍼티를 포함하는 클래스는 꼭 `클래스 선언 부분에 abstract 예약어`가 있어야 합니다.

```kotlin
abstract class AbstractTestClass {
    fun myFun1(){
        // ...
    }
    abstract fun myFun2()       // 추상 함수를 포함

    abstract val data1 : String // 추상 프로퍼티 포함
}
```

### 추상 클래스 이용

 - 추상 클래스는 객체를 생성할 수 없으며, 추상 클래스를 `상속받는 하위 클래스의 객체를 생성해서 이용` 해야 합니다.

 - 추상 클래스는 선언에 open 예약어를 명시하지 않아도 다른 클래스가 상속할 수 있습니다.

    - 추상 클래스를 상속받는 클래스는 는 상위 추상 클래스에 정의된 추상 함수와 추상 프로퍼티를 모두 `재정의 해야 합니다`.


보통 공통된 함수를 사용하지만 구체적인 정의는 따로 작성해야할 경우 abstract를 이용해 무조건 재정의 하게끔 사용 합니다.

ex)
```kotlin
abstract class Human {
    abstract var name: String
    abstract fun sayNameAndHi()
}

class Lee: Human() {
    override var name = "lee"
    override fun sayNameAndHi() {
        println("hi my name is $name")
    }
}

class Kim: Human() {
    override var name = "kim"
    override fun sayNameAndHi() {
        println("hi i'm $name")
    }
}

// ...
Lee().sayNameAndHi()
Kim().sayNameAndHi()
```

결과
```
hi my name is lee
hi i'm kim
```

## 인터페이스 

 > 객체 생성이 불가능 하며 `추상 함수를 선언하는 것이 주목적` 입니다.

추상 클래스는 `객체를 정의할 목적(is a)`이며 객체가 포함하는 행위 중 하위에서 재정의해서 사용할 함수만 추상형으로 명시하고<br> 인터페이스는 객체를 정의할 목적이 아니라 어떤 객체가 되든 그런 `행동을 가진다는 것(has a)을 명시`하기 위해 사용 됩니다.

### 인터페이스 선언 및 구현

 - 인터페이스는 객체로 생성할 수 없습니다.

 - 인터페이스는 클래스가 아니므로 `생성자가 없으며 구현하는 클래스는 콜론 : 을 이용`합니다. 

 - 인터페이스의 프로퍼티, 추상 함수는 명시하지 않아도 `기본으로 abstract`가 적용됩니다.

 - 인터페이스에는 구현 내용이 있는 함수를 정의 할수도 있으며 그대로 이용할 수 있습니다.

 - 클래스에서 인터페이스를 구현할 때 `여러 인터페이스를 나열해 한꺼번에 구현할 수 있습니다.`

형식 ex)
```
class 클래스명 : 인터페이스1
```
 
ex)
```kotlin
interface Human {
    var name: String
    fun sayNameAndHi()
    fun justHi(){
        println("Hi")
    }
}

class Lee : Human {
    override var name = "lee"
    override fun sayNameAndHi() {
        justHi()
        println("hi my name is $name")
    }
}

class Kim : Human {
    override var name = "kim"
    override fun sayNameAndHi() {
        println("hi i'm $name")
    }
}

// ...
val lee = Lee()
val kim = Kim()

lee.sayNameAndHi()
kim.justHi()
kim.sayNameAndHi()
```

결과
```
Hi
hi my name is lee
Hi
hi i'm kim
```

#### **클래스에서 여러 인터페이스 구현**

 - 클래스에서 `이러 인터페이스를 한꺼번에 구현`할 수 있습니다

ex)
```kotlin
interface MyInterface1 { 
    fun myFun1()
}
interface MyInterFace2 {
    fun myFun2()
}
class MyClass : MyInterface1, MyInterFace2 {
    override fun myFun1() { }
    override fun myFun2() { }
}
```

#### **클래스에서 상속과 인터페이스 혼용**

 - `상속받을 클래스는 하나만` 명시할 수 있으며, `인터페이스는 여러개` 구현할 수 있습니다

ex)
```kotlin
open class Super { }
interface MyInterface1 { 
    fun myFun1()
}
interface MyInterFace2 {
    fun myFun2()
}

class Sub : Super(),MyInterface1,MyInterFace2 {
    override fun myFun1() { }
    override fun myFun2() { }
}
```

### 오버라이드 함수 식별

 - 상위 클래스에 정의된 함수명과 인터페이스에 정의된 `함수명이 중복되는 경우 함수를 식별해서 이용해야` 합니다.

#### **같은 이름의 추상 함수가 여러 개일 때**

 - 재정의 해야하는 추상 함수는 여러 개인데 이름이 모두 같은 경우 한번만 함수를 재정의해주면 됩니다.

ex)
```kotlin
interface MyInterFace1 {
    fun funA()
}
interface MyInterFace2 {
    fun funA()
}

abstract class Super {
    abstract fun funA()
}

class Sub : Super(),MyInterFace1,MyInterFace2{
    override fun funA() {
        // ...
    }
}
```

#### **같은 이름으로 구현된 함수가 여러 개일 때**

 - 같은 이름으로 구현된 함수가 여러 개인 경우 해당 함수를 꼭 override 해야 합니다.

 - 인터페이스명을 명시해 사용할 수 있습니다.

   -  `super<인터페이스명>.함수명`

ex)
```kotlin
interface MyInterface1 { 
    fun funA(){
        // ...
    }
}
interface MyInterface2 { 
    fun funA(){
        // ...
    }
}
class MyClass : MyInterface1,MyInterface2 {
    override fun funA(){
        super<MyInterface1>.funA()
        super<MyInterface2>.funA()
    }
}
```




---

참조 : 깡샘의 코틀린 프로그래밍
