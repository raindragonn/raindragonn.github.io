---
layout: "post"
title: "[kotlin] kotlin의 상속"
subtitle: "kotlin 상속"
date:       2021-04-23
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
    - 상속
---

## 상속

> 상속이란 클래스를 선언할 때 다른 `상위 클래스를 참조해 작성`하는 개념 입니다.<br>
> 상위 클래스에 정의된 멤버(함수, 프로퍼티)를 하위 클래스(상속받아 정의 하는 클래스)에서 `자신의 멤버처럼 사용`할 수 있습니다.

### `Any` 클래스

 - 코틀린의 모든 클래스는 Any 클래스의 서브 클래스 입니다.

    - 클래스를 선언할 때 `상위 클래스를 명시하지 않으면 기본으로 Any 클래스를 상속` 받습니다.

 - Any 클래스는 자바의 java.lang.Object 클래스와 다릅니다.

 - Any 클래스 내부에는 `equals(), toString(), hashCode()` 이외의 다른 멤버들은 제공 하지 않습니다.

ex)
```kotlin
class User { } // 암시적으로 Any를 상속받음
// class User : Any() { } 와 같다

// ...
val user : Any = User()
```

### 상속을 통한 클래스 정의

 - 상속 받으려면 상위 클래스에 상속 허용 여부를 결정하는 예약어 `open` 예약어가 명시 되어 있어야 합니다.

    - 상속 허용 여부를 명시하지 않으면 기본으로 final로 적용 됩니다.

    - 자바의 경우 final 로 명시해야 상속 불가능, 코틀린의 경우 open으로 명시해야 상속 가능

 - `상속 관계 표현은 콜른 ( : )을 이용` 합니다.

 - 하위 클래스의 객체를 생성할 때 어떤 식으로든 `상위 클래스의 생성자는 무조건 실행 되어야 합니다.`


여러 클래스에서 `공통으로 필요로 하는 속성(property)과 행동(function)을 상속을 통해 상위 클래스로 분리`할 수 있습니다.

ex) 
```kotlin

open class Shape(val x: Int, val y: Int, val name: String) {
    fun print(){
        println("$name : location : $x,$y")
    }
}

class Rect(x: Int, y: Int, name: String = "사각형") : Shape(x, y, name) //하위 클래스의 생성자를 받아 상위 클래스의 주 생성자 호출
class Circle(x: Int, y: Int, name: String = "원형") : Shape(x, y, name)

// ...

val rect = Rect(10,20)
val circle = Circle(20,30)
rect.print()
circle.print()     
```

결과
```
사각형 : location : 10,20
원형 : location : 20,30
```

### 오버라이드 (Override)

 - 오버라이드는 상위 클래스에 정의된 프로퍼티나 함수를 `하위 클래스에서 재정의` 하는 것을 말합니다.

#### **함수 오버라이드**

 - 상위 클래스에 정의된 함수를 그대로 이용하지 않고, 다시 정의해야 할 때는 클래스와 마찬가지로 `open` 예약어를 통해 해당 함수가 `재정의할 수 있음을 명시`해야 합니다.
 
 - 하위 클래스에서 상위 클래스의 함수를 재정의할 때는 `함수 앞에 override 예약어를 추가`해야 합니다.

 - 만약 하위 클래스에서 `또다시` 하위 클래스를 작성한다면 `open이 아닌 override 예약어`가 하위클래스에서의 재정의를 `암묵적으로 허용`합니다.

   - 상위 클래스의 함수를 재정의 했지만, 하위 클래스에서는 자신을 재정의 하지 못하도록 하려면 final 예약어를 이용하면 됩니다.
   
   - ex) `final override fun print(){...}`
  


ex) 
```kotlin
open class Shape(val x: Int, val y: Int, val name: String) {
    open fun print(){
        println("$name : location : $x,$y")
    }
}

class Rect(x: Int, y: Int, name: String = "사각형") : Shape(x, y, name){
    override fun print() {
        println("Rect($x,$y,$name) call")
    }
}
class Circle(x: Int, y: Int, name: String = "원형") : Shape(x, y, name){
    override fun print() {
        println("Circle($x,$y,$name) call")
    }
}

// ...

val rect = Rect(10,20)
val circle = Circle(20,30)
rect.print()
circle.print()     
```

결과
```
Rect(10,20,사각형) call
Circle(20,30,원형) call
```

#### **프로퍼티 오버라이드**

 - 함수의 오버라이드와 같은 개념 입니다.

 - 상위 클래스의 프로퍼티에 재정의할 수 있음을 명시하기 위해 `open` 예약어를 사용해야 합니다.
  
 - 하위 클래스의 프로퍼티는 `override` 예약어로 이 프로퍼티는 상위 클래스의 프로퍼티를 재정의한 것이라고 명시 해야 합니다.
 
`프로퍼티를 오버라이드할 때의 규칙`

 - 상위 클래스의 프로퍼티와 `이름, 타입이 모두 같아야` 합니다.

 - 상위 클래스에 `val`로 선언된 프로퍼티는 하위에서 `val, var로` 재정의 가능 합니다.

 - 상위 클래스에 `var`로 선언된 프로퍼티는 하위에서 `var로만` 재정의 가능 합니다.

 - 상위 클래스에 프로퍼티가 `nullable한 타입인경우 nonNull 타입으로 재정의 가능` 합니다.

 - 상위 클래스에 프로퍼티가 `nonNull한 타입인 경우 하위에서 nullable한 타입으로는 재정의 불가` 합니다.

ex)
```kotlin
open class Super{
    open val name : String = "Lee"
    open var age : Int = 27
    open val email : String? = null
    open val address : String = "Seoul"
}

class Sub : Super(){
    override var name : String = "jo"  
    override val age : Int = 20             // 에러
    override val email : String = "a@a.com"
    override val address : String? = null   // 에러
}

```

#### **상위 클래스 멤버 접근 (Super)**

 - 하위 클래스에서 재정의한 멤버가 있어도 상위 클래스에 정의한 멤버도 함께 이용 할때 `super`를 사용합니다.


ex)
```kotlin
open class Parent {
    open var name : String = "parent"
    open fun someFun(){
        println("super..someFun")
    }
}
class Child : Parent(){
    override var name = "자식"
    override fun someFun(){
        super.someFun()     // super로 상위 클래스의 재정의 되지 않은 someFun 호출
        println("child..someFun ${super.name}")     // super로 상위 클래스의 재정의 되지 않은 parent 출력
    }
}
// ...
val child = Child()
child.someFun()
```

결과
```
super..someFun
child..someFun parent
```

### 상속과 생성자

#### **상위 클래스 생성자 호출**

 - 하위 클래스의 객체를 생성할 때 어떤 식으로든 `상위 클래스의 생성자는 무조건 실행되어야 합니다.`

 - 클래스의 주 생성자가 선언되어 있다면 해당 클래스의 보조 생성자에서는 `주 생성자와 연결하기 위한 this()구문이 추가`되어야 합니다.

**상위 클래스에 명시적으로 생성자가 선언된 경우**

ex)
```kotlin
open class Super(name: String)
class Sub : Super() // 에러_상위 클래스의 생성자를 실행 하지만 상위 클래스에는 매개변수없는 생성자가 없습니다.
class Sub() : Super("Lee") // 정상
class Sub(name) : Super(name) // 정상

open class Super1(name : String){
    constructor(name: String, age: Int) : this(name){} // 보조생성자 에서 this로 주 생성자 호출
}
class Sub1 : Super{
    constructor(name: String) : super(name) { }     // 상위 클래스의 주 생성자로 호출
    constructor(name: String, age: Int) : super(name,age)   // 상위 클래스의 보조 생성자로호출
}
```

#### **상하위 생성자의 수행 흐름**

 1. this() 혹은 super()에 의한 다른 생성자 호출
 2. init 블록 호출
 3. 생성자의 {} 영역 실행

ex)
```kotlin
open class Super{
    constructor(name: String, age: Int) {
        println("Super constructor($name,$age)")
    }
    init {
        println("Super init")
    }
}
class Sub(name: String) : Super(name,10) {
    constructor(name: String, age: Int) : this(name) {
        println("Sub constructor($name,$age)")
    }
    init {
        println("Sub init")
    }
}

// ...

Sub("Lee") // 
println("--------")
Sub("Lee",27)
```

결과
```
Super init
Super constructor(Lee,10)
Sub init
--------
Super init
Super constructor(Lee,10)
Sub init
Sub constructor(Lee,27)
```

### 상속과 캐스팅

 - `캐스팅이란 형 변환을 의미 합니다.` (Int 타입의 객체가 Double 타입으로 변환하는 등 데이터 타입의 변환)

 - `기초 데이터 타입의 캐스팅`은 자동 형 변환이 안되고 `함수를 이용`해야 합니다.

    - `val double : Double = 10.toDouble()`

#### **스마트 캐스팅**

 > 스마트 캐스팅이란 개발자가 명시작으로 캐스팅 하지 않아도 `자동으로 캐스팅 되는 것`을 의미합니다.

**`is` 예약어 이용시**

 - 코틀린에서는 `is라는 연산자를 이용해 타입을 확인`할 수 있습니다.
 
 - 조건문에서 `is 연산의 반환값이 true인 경우 해당 타입은 스마트 캐스팅` 됩니다.


ex)
```kotlin
fun smartCast(data: Any): Int =
    if (data is Int)        
        data * data     // is연산자로 data의 타입이 Int인 경우 스마트 캐스팅 발생
    else
        0

// ...

println("result = ${smartCast(10)}") 
println("result = ${smartCast(10.0)}")
// 출력
// result = 100
// result = 0
```

**`as` 예약어 이용시**

 - 코틀린에서는 상속 관계에 있는 객체를 `명시적으로 캐스팅할 때 as`를 이용합니다.

 - null 허용 객체의 캐스팅의 경우 `as?` 를 이용합니다

 - `as?` 정상적인 객체가 대입되면 정상적으로 캐스팅, `null이 대입되면 null을 반환` 합니다.

형식
```
객체 as 타입
```


---

참조 : 깡샘의 코틀린 프로그래밍
