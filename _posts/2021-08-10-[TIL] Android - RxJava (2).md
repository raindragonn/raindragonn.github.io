---
layout: "post"
title: "[TIL] Android - RxJava (2) 연산자"
subtitle: "ReactiveX"
date:       2021-08-11
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
    - RxJava
    - ReactiveX
---

## RxJava 연산자

RxJava에서는 **특정한 작업을 수행하는 메서드**를 연산자라고 합니다.대부분의 Observable 연산자는 Observable을 반환하므로 연산자의 결과가 다음 연산자에 영향을 미치기 떄문에 Observable의 체이닝 순서는 중요 합니다.

### Observable을 생성하는 연산자

`just()`, `create()` 와 같은 Observable을 생성하는 연산자 종류 중 일부 입니다.

#### `defer()` 연산자

 - 옵저버가 구독할 때까지 Observable의 생성을 지연시키며, `subscribe()` 메서드를 호출할 때 Observable 아이템을 생성 합니다.
 - 구독하기 직전에 Observable을 생성하여 **가장 최신의 데이터를 얻는게 목적일때 사용** 됩니다.

#### `interval()` 연산자

- 주어진 시간 간격으로 순서대로 정수를 발행 합니다.
- 구독을 중지하기 전까지는 무한히 데이터를 발행하므로 **적절한 시점에 폐기하는 것이 중요**합니다.

#### `range(n, m)` 연산자

- **특정 범위의 정수를 순서대로 발행** 합니다.
- 첫번째 매개변수는 시작점, 두번째 매개변수는 아이템 갯수 입니다.

#### `timer()` 연산자

- **특정 시간 지연**시킨 뒤 아이템(0L)을 발행 합니다.
  
### Observable을 변형하는 연산자

#### `map()` 연산자

- 발행되는 **아이템을 변환하는 가장 기본적인 방법** 입니다.
- 발행되는 값에 대해 원하는 수식을 적용하거나 다른 타입으로 변환시킬 수 있습니다.

#### `flatMap()` 연산자

- Observable을 **또 다른 Observable로 변환**시킵니다.
- 변환시킨 Observable의 방출되는 아이템을 병합하여 방출시킵니다. 1:N 형태로 새로운 시퀀스가 발행됩니다.

#### `buffer(n)` 연산자

- Observable이 발행하는 **아이템을 묶어서 List로 변환** 합니다.
- 매개변수로 넘기는 정수는 List의 사이즈 입니다.

#### `scan()` 연산자

- 순차적으로 발행되는 `아이템들의 연산을 다음 아이템 발행의 첫 번째 인자로 전달`합니다.
- 두 개의 아이템이 있어야 하므로 첫 번째 아이템 발행 시에는 아이템이 변형되지 않고 그대로 발행됩니다.
- 누산기라고도 합니다.

#### `groupBy()` 연산자

- 아이템들을 특정 **그룹화된 GroupObservable로 재정의**할 수 있습니다.

### Observable을 필터링하는 연산자

Observable로부터 발행되는 아이템들을 선택적으로 발행하도록 하는 연산자 입니다.

#### `debounce()`연산자

- 특정 시간 동안 다른 아이템이 발행되지 않을 때만 아이템을 발행하도록 하는 연산자 입니다.
- **반복적으로 빠르게 발행된 아이템들을 필터링**할 때 유용 합니다.
- ex) 검색어 입력시 모든 문자의 입력마다 검색하지 않게 하기 위해

#### `distinct()` 연산자

- 이미 발행한 아이템을 **중복해 발행하지 않도록 필터링** 합니다.

#### `elementAt()` 연산자

- 발행되는 아이템 시퀀스에서 **특정 인덱스에 해당하는 아이템을 필터링** 합니다.

#### `filter()` 연산자

- **조건식이 true일 때만 아이템을 발행**하도록 합니다.

#### `sample()` 연산자

- **일정 시간 간격으로 가장 최근에** Observable이 배출한 아이템을 방출하는 연산자 입니다.

#### `skip(n)` 연산자

- Observable이 발행하는 **n개의 아이템을 무시하고 이후에 나오는 아이템을 발행**하는 연산자 입니다.

#### `take(n)` 연산자

- `skip(n)` 연산자와는 반대로 **처음 발행하는 n개의 아이템만 방행**하도록 하는 연산자 입니다.

### Observable을 결합하는 연산자

여러 Observable 소스를 결합하여 하나의 Observable을 생성하고, 동작하는 연산자 입니다.

#### `combineLatest()` 연산자

- 두 개의 Observable 중 한 소스에서 아이템이 발행될 때, **두 Observable에서 가장 최근에 발행한 아이템을 취합하여 하나로 발행** 합니다.
  
#### `zip()` 연산자

- 여러 Observable을 하나로 결합하여 지정된 함수를 통해 하나의 아이템으로 발행 합니다.
- `combineLatest()` 와 다르게 **발행순서를 지켜 아이템을 결합** 합니다.

#### `merge()` 연산자

- 여러 Observable이 발행하는 아이템을 **발행 시점에 하나의 스트림에 교차해 끼워 넣어 하나의 Observable을 만듭니다.**

### 오류를 다루는 연산자

#### `onErrorReturn()` 연산자

- 오류가 발생하면 아이템 발행을 종료 합니다.
- `onError()`를 호출하는 대신에 오류처리를 위한 함수를 실행 합니다.

#### `onErrorResumeNext()` 연산자

- 오류 발생 시 기존 스트림을 종료시키고, **다른 Observable소스로 스트림을 대체** 합니다.

#### `retry(n)` 연산자

- 오류 발생 시 **Observable을 재구독** 하도록 합니다.
- 매개변수를 비워둘 경우 무한히 재시도 하게 되고 스트림이 종료되지 않습니다.
- 적당한 재시도 횟수를 명시하는것이 좋습니다.


---

참고자료

[아키텍처를 알아야 앱 개발이 보인다](http://www.yes24.com/Product/Goods/89958199)