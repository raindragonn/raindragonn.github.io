---
layout: "post"
title: "[Android] Android Context란?"
subtitle: "Android Context"
author: "raindragonn"
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
    - Android
    - View
    - Context
---

# Context 란?

안드로이드 개발을 하다 보면 자주 나오는 Context는 무엇일까요?

[안드로이드 공식 레퍼런스][Android Reference] 에서는 이렇게 나와있습니다.

> Interface to global information about an application environment. This is an abstract class whose implementation is provided by the Android system. It allows access to application-specific resources and classes, as well as up-calls for application-level operations such as launching activities, broadcasting and receiving intents, etc.

> 애플리케이션 환경에 대한 글로벌 정보에 대한 인터페이스입니다. 이것은 안드로이드 시스템에서 구현이 제공되는 추상 클래스이다. 애플리케이션별 리소스 및 클래스에 대한 액세스는 물론, 액티비티 시작, 브로드캐스팅, 인텐트 수신 등과 같은 애플리케이션 수준의 운영을 위한 호출을 할 수 있다.

요약 하자면

`Context를 이용하여 어플리케이션의 자원이나 클래스에 접근 및 시스템 레벨의 정보를 얻을수 있다.`

조금더 자세히 보자면 `안드로이드 시스템 에서는 프로세스`를 관리를하고 

`어플리케이션 관리는 ActivityManagerService 객체 에서` 하고 있으며,

이 객체는 각각의 앱을 key-value값의 쌍으로 관리 하기 때문에

`Context`를 통해 필요한 정보를 제공 하여 접근 할수 있습니다.


## Context의 종류

크게 두가지로 분류를 하자면 2가지 입니다.

`1. Application Context`
- 싱글톤으로 Application의 생명주기와 함께하는 Context 입니다

`2. Activity Context`

- 액티비티의 생명주기와 함께하는 Context 입니다.
- 메모리 릭에 주의 해야합니다.

## Context 참조 하는법

`View.getContext()`

- view를 생성할때 생성자의 인자로 넣어준 Context가 반환됩니다.
- 일반적으로 Activity에서 view를 띄우기 때문에 Activity의 Context가 반환됩니다.
  
`getApplicationContext()`

- Application Context를 반환 합니다.

`getBaseContext()`

- ContextWrapper의 Context를 반환 합니다.
- 일반적으로 Activity Context 입니다.


## Context 사용시 주의 할점

`Context를 참조 하는 작업에는 Context의 Scope를 잘 생각하고 사용 해야 합니다.`

예시로 어플리케이션 전반에 사용되는 싱글톤 객체가 있다고 했을때

Activity Context를 참조 후 해당 Activity가 종료 된다면 `GC(가비지 콜렉팅)되지 않으므로 메모리 누수가 발생`할 수 있습니다.


---
참고 자료

[찰스님 블로그](https://www.charlezz.com/?p=1080)

[안드로이드 - 4대 컴포넌트, 생명주기, 컨텍스트, 인텐트](https://stickode.com/detail.html?no=1755)

[Yub's님 블로그](https://velog.io/@jaeyunn_15/Android-Context)


[Android Reference]: https://developer.android.com/reference/android/content/Context