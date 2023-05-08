---
layout: "post"
title: "[kotlin] kotlin의 다양한 클래스"
subtitle: "data, Enum, Sealed, Object class"
date:       2021-04-26
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


## data class

 > 흔히 객체지향 프로그래밍에서 `VO(Valiew-Object)클래스`라 부르는 클래스 내부에 특별한 로직의 함수 없이 데이터만을 포함하는 클래스로 `더 편하게 이용하기 위한 클래스` 입니다.

 - class 앞에 `data`라는 예약어로 선언합니다.
 
 - 주 생성자를 선언해야 하며 `주 생성자의 매개변수는 최소 하나 이상`이어야 합니다.

 - 모든 주 생성자의 매개변수는 `var 혹은 val로` 선언해야 합니다.

 - 데이터 클래스는 abstract, open, sealed, inner등의 `예약어를 추가할 수 없습니다.`

ex)
```kotlin
data class User(
    val uid: Int,
    var name: String,
    var age: Int,
    var address: String
)
```

### 데이터클래스가 지원하는 함수

 - 데이터 클래스는 개발자가 클래스내에 선언하지 않아도 자동으로 다음의 함수를 제공합니다.

#### 1. `equals() / hashCode()`

 - `객체의 데이터가 같은지`를 비교하는 작업을 도와주는 함수 입니다.

    - data class가 아닌 Any 클래스의 equals() 함수는 data 클래스의 equals()와는 다르게 객체를 비교하는 함수 입니다.

 - 주 생성자의 매개변수가 똑같다고 하더라고 `각기 다른 data class일 경우 false`를 반환 합니다.

 - 주 생성자의 매개변수가 아닌 클래스 내의 프로퍼티의 값은 비교 대상이 아닙니다.


ex)
```kotlin
class Human(val name: String, val age: Int) // 일반 클래스(기본적으로 Any를 상속 받는다)
data class DataHuman(val name: String, val age: Int){ // data class
    var email : String = "a@a.com"
}

val human1 = Human("Lee", 27)
val human2 = Human("Lee", 27)

val dataHuman1 = DataHuman("Lee",27)
val dataHuman2 = DataHuman("Lee",27)

val dataHuman3 = DataHuman("Lee",27)
val dataHuman4 = DataHuman("Lee",27).apply {
    email = "b@b.com"
}

// ...


// 일반적인 클래스(Any 클래스)의 equals() 는 객체를 비교 하기 때문에 false
println(human1.equals(human2))

// data class 는 주 생성자의 데이터 값을 비교 하기 때문에 true
println(dataHuman1.equals(dataHuman2))

// data class 는 내부 프로퍼티의 값은 다르지만 주 생성자의 데이터 값만 비교 하기 때문에 true
println(dataHuman3.equals(dataHuman4))
```

결과
```
false
true
true
```
    

#### 2. `toString()`
 
 - `데이터를 문자열로 반환하는 함수` 입니다.

 - equals()와 마찬가지로 `주 생성자의 매개변수가 아닌 클래스 내의 프로퍼티의 값은 대상이 아닙니다.`

    - equlas(), hashCode(), toString()은 재정의(override)가 가능하므로 개발자가 원한다면 `재정의를 통해 프로퍼티 의값도 문자열로 반환할 수 있습니다.`

ex)
```kotlin
data class User(val uid:Int, val name: String){
    var email: String = "a@a.com"
}

data class Human(val age:Int, val name: String){
    var email: String = "a@a.com"
    override fun toString(): String {
        return "Human(age=$age, name=$name){ email=$email }"
    }
}

// ...

val user = User(1,"Lee")
val human = Human(2,"Lee2")

println(user.toString())        // 주 생성자에 대한 문자열 반환
println(human.toString())       // toString 의 재정의를 통한 문자열 반환
```

결과

```
User(uid=1, name=Lee)
Human(age=2, name=Lee2){ email=a@a.com }
```



#### 3. `componentN()`

 - 데이터 클래스의 `프로퍼티의 이름이 아니라 componentN() 함수를 이용하여` 해당 프로퍼티 값을 가져올 수 있습니다. 

    - 주 생성자의 프로퍼티 수에 따라 componentN() 함수가 자동으로 만들어집니다.
    
    - ex)
    
    - 주 생성자의 프로퍼티가 `2개라면 component1(), component2()`

    - 주 생성자의 프로퍼티가 `3개라면 component1(), component2(), component3()`

 - 구조 분해 선언(Destructuring declarations)을 할 수 있습니다.

    - 구조 분해 선언이란 복합적인 값을 분해해서 여러 다른 변수에 나눠 담는 것을 말합니다.

ex)
```kotlin
data class User(val uid: Int, val name: String)

fun main(){
    val user = User(1, "Lee")
    val (userId, userName) = user // 데이터 클래스의 componentN()을 통해 각각 프로퍼티 값 대입

    println(userId.toString())
    println(userName)
}
```

결과

```
1
Lee
```

#### 4. `copy()`

 - 객체를 복사해서 다른 객체를 만들어주는 함수 입니다.

 - `데이터의 일부분만 변경해서 다른 객체를 만들 때 유용`합니다.

ex)
```kotlin
data class User(val uid: Int, val name: String)

// ...
var user = User(1, "Lee")
var user2 = user.copy(uid = 2)

println(user.toString())
println(user2.toString())
```

결과

```
User(uid=1, name=Lee)
User(uid=2, name=Lee)
```

## Enum 클래스

 > 흔히 열거형 상수 클래스 라고 표현합니다. `상수 여러개를 대입해 선언하고 이 값중의 하나를 지정해서 사용`합니다.

 - class 앞에 `enum`이라는 예약어로 선언합니다.

 - 열거형 클래스에 속한 `상수`는 기본으로 `name과 original 프로퍼티`를 제공합니다

    - name은 열거 상수의 문자열이며, original은 순서를 나타내는 인덱스 번호 입니다.

 - 열거형 `클래스`는 기본으로 `values()와 valuesOf() 함수`를 제공합니다

    - values()는 열거한 상수 모두를 객체로 가져오는 함수 입니다.

    - valuesOf()는 인수로 전달한 문자열에 해당하는 열거 상수를 가져오는 함수 입니다.


ex)
```kotlin
enum class Direction{
    NORTH, SOUTH, WEST, EAST
}

fun main(){
    val direction : Direction = Direction.SOUTH
    println("I'm in ${direction.name}, index = ${direction.ordinal}")

    val arr : Array<Direction> = Direction.values()
    println("-----")
    arr.forEach {
        println(it.name)
    }
    println("-----")
    val selectDirection = Direction.valueOf("SOUTH")
    println(selectDirection.name)
}
```

결과
```
I'm in SOUTH, index = 1
-----
NORTH
SOUTH
WEST
EAST
-----
SOUTH
```

### 열거 상수에 데이터 대입하기

 - 열거 상수에 데이터를 대입하려면 열거형 클래스를 선언할 때 `주 생성자의 매개변수에 프로퍼티를 선언`해야 합니다.

 - 주 생성자의 매개변수에 선언된 `프로퍼티의 이름으로 접근`할 수 있습니다.

ex)
```kotlin
enum class Direction(val no: Int){
    NORTH(10),
    SOUTH(20),
    WEST(30),
    EAST(40)
}

// ...
println(Direction.NORTH.no)
```
결과
```
10
```

### 익명 클래스 이용하기

enum class에 열거한 상수는 해당 `enum class를 상속받는 클래스의 객체`입니다.

이때 서브 클래스의 이름은 없으며 이러한 클래스를 `익명 클래스(Anonymous class)` 라고 합니다.

예를들어 `enum class Direction{ NORTH }` 라고 작성시

NORTH는 Direction 클래스를 상속받는 클래스의 객체입니다.

따라서 상위 클래스가되는 Direction의 생성자에 맞추어 호출해야 하므로 

`enum class Direction(val no : Int){ NORTH(10) }`의 경우 NORTH에도 생성자 `(10)` 가 필요합니다.

**`하지만 익명 클래스를 직접 정의하면 주 생성자와 상관없이 임의의 프로퍼티나 함수를 추가 할 수 있습니다`**

 - 이때 `주의 사항`은

    - 상수는 콤마(`,`)로 구분하므로 프로퍼티나 함수와 구분해 주어야하는데

    - 세미콜론(`;`)를 통해 `열거상수와 클래스에 선언하는 프로퍼티와 함수를 구분`해 주어야합니다.

ex)

```kotlin
enum class Direction{
    NORTH{                  // 익명 클래스를 정의 하는 부분
        override var data1: Int = 10

        override fun printData() {
            println("NORTH data1 = $data1")
        }
    },
    SOUTH{
        override var data1: Int = 20

        override fun printData() {
            println("SOUTH data1 = $data1")
        }
    };          // 세미콜론으로 영역을 구분해야한다. 앞 부분이 열거 상수, 뒷부분이 클래스에 선언하는 프로퍼티와 함수

    abstract var data1: Int         // 추상형 프로퍼티, 함수로 재정의를 강제
    abstract fun printData()
}

// ...
val south = Direction.SOUTH

south.data1 = 100
south.printData()
```

결과
```
SOUTH data1 = 100
```

## Sealed 클래스

 > 영어 단어 뜻 그대로 몇몇 타입을 묶기 위한 용도로 사용합니다.

- 클래스 앞에 `sealed` 예약어로 선언합니다.

- enum과 비슷하지만 가장 큰 차이는 `서로 다른 생성자(즉 서로 다른 프로퍼티와 함수를 갖을 수있습니다.)를 갖는다는 점` 입니다

- Sealed class는 abstract 클래스로, `객체로 생성할 수 없습니다.`

- Sealed class의 `생성자는 private입니다.` public으로 설정할 수 없습니다.

- Sealed class와 그 하위 클래스는 `동일한 파일에 정의되어야 합니다.` 서로 다른 파일에서 정의할 수 없습니다.

- 하위 클래스는 class, data class, object class으로 정의할 수 있습니다.


ex)

```kotlin
sealed class Color {
    object Red : Color()
    object Green : Color()
    object Blue : Color()
}

fun printColor(color: Color) {
    when (color) {
        is Color.Blue -> println("it's Blue")
        is Color.Green -> println("it's Green")
        is Color.Red -> println("it's Red")
    }
}

fun main(){
    val color = Color.Blue
    printColor(color)
}
```

결과
```
it's Blue
```

## Nested class (중첩 클래스)

 > 어떤 클래스 내부에 정의한 클래스를 흔히 inner 클래스라 부르지만 코틀린에서는 inner 라는 예약어가 따로 있으며 `특정 클래스 내에 선언된 클래스`를 Nested class라 지칭합니다.

 - 기본적으로 Nested 클래스에서 외부 클래스(해당 클래스를 선언한 위치의 클래스)의 `멤버에 접근할 수 없습니다.`

 - 외부 클래스(해당 클래스를 선언한 위치의 클래스)의 멤버에 접근하기 위해서는 클래스 앞에 `inner`라는 예약어를 추가해야 합니다.

 - `inner` 클래스의 경우 외부에서 객체를 생성할 수 없으며 외부 클래스에서만 객체를 생성해야 합니다.

 ex)
 ```kotlin
class Outer {
    private var no: Int = 10

    fun outerFun() {
        println("it's outerFun()")
    }

    class Nested {
        val name: String = "Lee"
        fun myFun() {
            println("Nested.myFun() call")
            no = 20 // inner 클래스가 아니라 접근할 수 없다.
        }
    }

    inner class InnerNested {
        val name: String = "Lee"
        fun myFun() {
            no = 20 // 정상 작동
            println("InnerNested.myFun() call")
        }
    }
}

fun main() {
    val obj = Outer.Nested()            // 일반적인 Nested class의경우 Outer의 객체로 접근 하지않는다.
    obj.myFun()
    val innerObj = Outer().InnerNested() // inner클래스의 경우 Outer의 객체로만 생성이 가능하다.
    innerObj.myFun()
}
 ```

## Object 클래스

 > class 대신 `object` 예약어로 정의한 클래스는 싱글턴(Singleton) 패턴이 적용되어 객체가 한번만 생성되도록 합니다.

 - 객체 생성이 클래스 선언과 동시에 이러우지므로 생성자를 선언할 수는 없습니다.

 - 여러번 호출 하더라도 객체는 한번만 생성이 됩니다.

ex)

```kotlin
object CoffeeFactory {
    val coffees = mutableListOf<Coffee>()

    fun makeCoffee(milk: Int): Coffee {
        val coffee = Coffee(milk)
        coffee.add(coffee)
        return coffee
    }
}
class Coffee(val milk: Int)

// ...

val coffee : Coffee= CoffeeFactory.makeCoffee(10) // CoffeeFactory의 함수로 Coffee 객체 생성
println(CoffeeFactory.coffes.size) //CoffeeFactory의 프로퍼티에 접근
```

### companion object

 - companion은 object와 함께 사용되어 object클래스의 멤버를 `object 클래스명을 사용하지 않고도`이용하게 해줍니다.

 - 자바의 static같은 효과를 낼 수 있습니다.

ex)
```kotlin
class Outer {
    companion object NestedClass {
        val no: Int = 0
        fun myFun() { }
    }
}

// ...

Outer.NestedClass.no
Outer.NestedClass.myFun()
Outer.no
Outer.myFun()
)
```

### 익명 객체로 사용하기

 - 익명객체는 이름이 없는 객체로, 한번만 사용되고 재사용되지 않을 때 사용합니다. 재사용되지 않기 때문에 귀찮게 클래스 이름을 지어주지 않는 것 입니다.

 - `class 클래스명 : 타입` 형태로 선언하는 대신 `object : 타입` 형태로 사용 할 수 있습니다.

ex)
```kotlin
interface Doing{
    fun doSomething()
}

class Do :Doing {
    override fun doSomething(){
        // ...
    }
}

fun start(doing: Doing) {
    doing.doSomething()
}

// ...
start(object: Doing{                // 익명 객체로 사용
    override fun doSomething(){
        // ...
    }
})

val do : Doing = Do()
start(do)                           // 클래스작성, 객체 생성 후 사용
```



---

참조 : 깡샘의 코틀린 프로그래밍

[Kotlin - object와 class 키워드의 차이점](https://codechacha.com/ko/kotlin-object-vs-class/)