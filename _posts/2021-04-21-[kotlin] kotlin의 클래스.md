---
layout: "post"
title: "[kotlin] kotlin의 클래스"
subtitle: "kotlin class"
date:       2021-04-21
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
---

## class

> 객체지향 프로그래밍의 기본은 클래스를 선언하고 클래스 내에 여러 구성요소를 담은 후 객체로 생성해서 사용하는 것입니다.

### 클래스 선언

코틀린에서 클래스를 선언하는 방법은 `class 예약어`를 이용합니다.

코틀린 소스는 확장자가 kt인 파일로 만들어지는데 그 파일 안에 클래스를 `다양한 형태`로 정의해서 사용 할 수 있습니다.(아래 그림 참고)

클래스에는 대표적으로 프로퍼티, 메서드, 함수, 생성자, 클래스 등을 선언할 수 있습니다.


![클래스형태][class]

[이미지출처][이미지출처]


### 객체 생성

>클래스의 구성요소를 이용하려면 구성요소가 메모리에 올라와야 하는데 `객체를 생성할 때 메모리가 할당`됩니다.<br>
따라서 객체 생성은 결국 클래스를 이용하기 위해 메모리를 할당하는 작업이며,
할당된 `메모리를 객체명으로 참조해서 이용`하는 개념입니다.

객체를 생성하면 그에 해당하는 메모리가 할당 되기에, 객체를 여러 개 생성하면 각기 다른 메모리를 참조 하므로 서로 다른 데이터를 유지하게 됩니다.



ex)
```kotlin
class Human{
    var name : String = "name"

    fun printName(){
        println("name = ${name}")
    }
}


fun main(){
    val test1 = Human()
    val test2 = Human()
    test1.name = "김씨"
    test2.name = "이씨"

    test1.printName()
    test2.printName()
}
```

결과 
```
name = 김씨
name = 이씨
```

### 생성자 (constructor)

> 생성자는 객체가 생성될 때 호출되는 함수이며, 코틀린에서는 `주 생성자`와 `보조 생성자`로 구분 합니다.

#### 주 생성자 

 - `클래스 선언 부분에 작성` 합니다.
 
 - `하나의 클래스에서 하나만` 정의할 수 있습니다.

 - 꼭 작성해야 하는 건 아니며 보조 생성자가 있다면 작성하지 않을 수 있습니다.

 - 개발자가 별도로 선언한 생성자가 없다면 컴파일러가 자동으로 매개변수가 없는 주 생성자를 추가합니다.

 - 생성자에 별도의 수식구문(어노테이션, 접근 제한자 등)이 없다면 constructor 예약어는 생략할 수 있습니다.

ex)
```kotlin
class TestClass1{ }

class TestClass2() { }

class TestClass3 constructor() { }

fun main(){
    val test1 = TestClass1()
    val test2 = TestClass2()
    val test3 = TestClass3()
}
```

**매개변수가 있는 주 생성자**

 -  주 생성자를 정의할 때 `매개변수를 선언하면` 해당 생성자로 객체를 생성할 때 `매개변수에 맞는 인수를 전달`해야 합니다.

 - 일반 함수처럼 생성자의 매개변수에도 `기본값`을 명시할 수 있습니다.
  
   -  객체를 생성할 때 해당 매개변수 값은 대입하지 않아도 됩니다.(기본값 적용)



ex)
```kotlin
class User1 constructor(name: String, age: Int){ }
class User2(name: String, age: Int){ }
class User3(name: String = "Lee", age: Int = 10){ }

//...

val user1 = User1() // 에러
val user2 = User1("lee", 27)
val user3 = User2("jo", 25)

val user4 = User3() // 기본값 적용
```

**생성자 초기화 블록 `init`**

 - 클래스의 생성자에 매개변수를 지정하는 것 이외에 `실행문을 작성`할 수도 있습니다.

   - 객체 생성과 동시에 init블록도 함께 실행 됩니다. 

ex)
```kotlin
class User1(name: String, age: Int){
    init{
        println("User1($name, $age)객체 생성")
    }
}

// ...

val user = User1("lee",27)
```

결과
```
User1(Lee, 27)객체 생성
```

**생성자 매개변수 값 이용**

 - 생성자의 매개변수로 받은 값은 클래스의 초기화 블록(init)이나 프로퍼티에서는 접근할 수 있지만, `클래스에 정의된 멤버 함수에서는 사용할 수 없습니다`.

 - 생성자의 매개변수를 멤버 함수에서 사용 하는 법으로는 2가지 있습니다
 
    1. 생성자의 매개변수를 클래스 `프로퍼티에 대입`하여 멤버 함수에서 사용하기
    
    2. 생성자 내에서 val, var를 이용해 매개변수를 `프로퍼티로 선언`하여 사용하기

ex)
```kotlin
class User1(name: String, age: Int){
    init{
        println("User1($name, $age)객체 생성")
    }
    val upperName = name.toUpperCase()

    fun test(){
        println(name) // Unresolved reference: name error
    }
}

// 1.생성자의 매개변수를 클래스 프로퍼티에 대입하여 멤버 함수에서 사용하기

class User2(name: String, age: Int) {
    init {
        println("User1($name, $age)객체 생성")
    }

    val mName = name

    fun test() {
        println(mName)
    }
}

// 2. 생성자 내에서 val, var를 이용해 매개변수를 프로퍼티로 선언하여 사용하기

class User3(val name: String, val age: Int) {
    init {
        println("User1($name, $age)객체 생성")
    }

    fun test() {
        println(name)
    }
}
```

 
#### 보조 생성자

 - 보조 생성자는 클래스 내부에 `constructor 예약어로 선언`합니다.

 - 보조 생성자를 선언했으면 주 생성자는 선언하지 않아도 됩니다.

 - 컴파일러는 보조 생성자나 주 생성자를 `선언하지 않았을 경우만` 매개변수 없는 주 생성자를 추가 합니다.

ex)
```kotlin
class User1 { } // 컴파일러가 매개변수 없는 주 생성자를 컴파일 단계에서 자동으로 추가 합니다.

class User2(name: String) { } // 주 생성자만 선언

class User3 { // 보조 생성자만 선언
    constructor(name : String) { }
}
```


**생성자 오버로딩**

 - 함수의 오버로딩과 마찬가지로 매개변수 부분을 다르게 해서 `여러 개의 생성자를 정의` 할 수 있습니다.

ex)
```kotlin
class User {
    constructor() {
        println("User() call")
    }

    constructor(name: String) {
        println("User($name) call")
    }

    constructor(name: String, age: Int) {
        println("User($name,$age) call")
    }
}
// ...
val user1 = User()
val user2 = User("Lee")
val user3 = User("Lee",27)
```

결과
```
User() call
User(Lee) call
User(Lee,27) call
```

**보조 생성자와 초기화 블록(init)**

 - 초기화 블록(init)과 보조 생성자를 함께 선언한 경우 `초기화 블록(init)이 먼저 실행되고 이후에 보조 생성자가 실행` 됩니다.
  

ex)
```kotlin
class User{
    init{
        println("init Call")
    }
    constructor(){
        println("constructor call")
    }
}
// ...
val user = User()
```

결과
```
init Call
constructor call
```

**주 생성자와 보조 생성자를 모두 선언**

 - 주 생성자를 선언 했다면 보조 생성자는 `this() 생성자를 이용해 직간접적으로 주 생성자에게 생성을 위임`해야만 합니다.

ex)
```kotlin
class User(name : String) {
    init {
        println("init call name = $name")
    }

//  Primary constructor call expected
//  constructor(name: String, age: Int) {
//      println("User($name,$age) call")
//  }
    constructor(name: String, age: Int) : this(name) {
        println("User($name,$age) call")
    }
}

// ...
val user1 = User("Lee")
val user2 = User("Lee",27)
```

결과
```
init call name = Lee
init call name = Lee
User(Lee,27) call
```






---

참조 : 깡샘의 코틀린 프로그래밍



[class]:/assets/images/post/2021-04-21-class.png 

[이미지출처]: https://github.com/kkangseongyun/kkangs_kotlin/blob/master/PDF/7%EC%9E%A5.%20%ED%81%B4%EB%9E%98%EC%8A%A4.pdf