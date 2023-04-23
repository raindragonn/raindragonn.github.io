---
layout: "post"
title: "[TIL] DI,의존성 주입"
subtitle: "Dependency Injection"
author: "raindragon"
banner:
  image: "/assets/images/base/banner.jpg"
  opacity: 0.618
  height: "38vh"
  min_height: "38vh"
  heading_style: "font-size: 2.25em; font-weight: bold; text-decoration: underline"
  subheading_style: "color: gold"
lang: ko
categories: [TIL - Android]
tags:
    - DI
    - Dependency Injection
---

# 의존성 주입(Dependency Injenction)

의존성을 주입한다.. 라는게 뭘까요? 일단 의존성이 뭔지 알아야곘죠?

## 1. 의존성이란 무엇일까?

```kotlin
class Keyboard{
    fun typing(){}
}


class Computer{
    val keyboard = Keyboard()

    fun memo(){
        //...
        keyboard.typing()
        //...
    }
}
```

해당 예제를 보면 Computer클래스로 메모를 하려할경우
Computer 클래스 내부 에서 새로운 객체(keyboard)를 만들어 사용 하기때문에 `Keyboard클래스에서 기능이 수정되거나 추가된다면 Computer 클래스에서도 해당 기능에 영향을 받게 됩니다.` 따라서 Computer 클래스는 Keyboaed 클래스에 의존성(결합도)이 높음을 알수있습니다.


## 2. 의존성 주입 이란?

위의 예제에서 Computer클래스에서 필요로하는 객체(keyboard)를 내부에서 생성하는것이 아닌, `외부에서 생성하고 받아와서 사용을 한다면`(사용 방식을 정의한 인터페이스에 대해서 알고) 어디선가 만든 의존성을 주입한다고 볼수 있겠습니다.   

>한마디로 정의 하자면   
*`두개 이상의 모듈이나 클래스간의 결합도를 낮추기 위해 외부에서 객체를 생성하고 주입하는것`*

```kotlin
class Computer(var keyboard: Keyboard) {
    // ...
}

```

### 의존성 주입의 필요성

#### 변경의 전이

 위의 예제와 같이 Computer 클래스는 Keyboard 라는 `한 가지 타입에 의존`합니다.

 이후에 사용자가 다른 키보드를 사용하는것을 원한다면 클래스명을 A_Keyboard로 변경하거나 새로 만들어야 합니다.

 해당 클래스를 의존하는 클래스 또한 `변경사항이 생길때 마다 같이 변경을 해줘야합니다.`

 이를 해결할 방법은 `해당 클래스(Computer)가 의존하는 Keyboard를 interface로 만드는 것입니다.`

 ```kotlin
 
 interface Keyboard { ... }

 class A_Keyboard : Keynoard { 
     // ...
 }

 class Computer {
     val keyboard = A_Keyboard()
     // val keyboaed = B_Keyboard
 }

 ```

#### 제어의 역전

 제어의 역전은 어떠한 일을 수행하도록 만들어진 프레임워크에 제어권을 위임함으로써

 관심사를 분리하는것을 의미합니다.

 `의존하는 타입의 객체의 생성 및 관리를 외부에 위임하는것`

```kotlin
class Test {
    fun main() {
        val keyboard = A_Keyboard()
        val computer = Computer(keyboard)
    }
}
```

