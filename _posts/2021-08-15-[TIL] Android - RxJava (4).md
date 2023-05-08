---
layout: "post"
title: "[TIL] Android - RxJava (4) 배압과 Flowable"
subtitle: "ReactiveX"
date:       2021-08-15
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

## 배압(BackPressure)

RxJavad에서 Observable은 생성자(Producer)와 소비자(Consumer)로 나눌 수 있습니다.

생성자는 아이템을 발행하며, 소비자는 생산자가 발행한 아이템을 구독합니다.

Observable에서 **아이템을 빠르게 발행하는데, 구독자는 이를 빠르게 소비하지 못한다면 생산한 아이템들이 메모리에 누적돠며, 이에 따라 계속해서 아이템을 발행한다면 메모리가 오버플로 될 것입니다.**

이런 현상을 `배압(BackPressure)`이라고 합니다. 이를 제어하지 못하면 OOM(OutOfMemmoryError)을 포함한 많은 문제를 발생시킬 수 있습니다.

## Flowable

RxJava에서는 배압을 직접 제어할 수 있는 솔루션인 Flowable을 제공 합니다.

Flowable은 배압을 스스로 조절하는 점이 Observable과의 차이점 입니다.

```kotlin
@Test
fun test() {
    Flowable.range(1, Int.MAX_VALUE)
        .map { item ->
            item.also {
                println("아이템 발행: $item")
            }
        }
        .observeOn(Schedulers.io())
        .subscribe { item ->
            Thread.sleep(100)
            println("아이템 소비 : $item")
        }
    Thread.sleep(30 * 1000L)
}
/**
 * 실행 결과
 * 아이템 발행: 1
 * 아이템 발행: 2
 * 아이템 발행: 3
 * ...
 * 아이템 발행: 126
 * 아이템 발행: 127
 * 아이템 발행: 128
 * 아이템 소비 : 1
 * 아이템 소비 : 2
 * 아이템 소비 : 3
 * ...
 * 아이템 소비 : 94
 * 아이템 소비 : 95
 * 아이템 소비 : 96
 * 아이템 발행: 129
 * 아이템 발행: 130
 * 아이템 발행: 131
 * ...
 */
```

실행 결과를 보면 첫 번째 발행은 1부터 128까지 이루어지고, 첫 번째 소비는 1부터 96까지 이루어 집니다.

위와 같은 발행과 소비의 차이가 있는 이유는 **다시 생산자가 발행하기까지 걸리는 시간으로 인해 소비자가 기다리는 일이 없도록 여유를 두기 위함입니다.**

하지만 시간을 기반으로 하는 interval 연산자와 Flowable을 같이 사용할 경우 문제가 발생할 수 있습니다.

**interval과 같은 연산자들은 스케줄러의 설정과 관계없이 시간을 기반으로 충실히 아이템을 발행합니다.**

그러므로 생산하는 쪽에서 블로킹 이슈가 발생하면 배압 제어 전략과 관계없이 MissingBackpressureException이 발생합니다.

```kotlin
@Test
fun test() {
    Flowable.interval(10, TimeUnit.MILLISECONDS)
        .observeOn(Schedulers.io())
        .map { item ->
            Thread.sleep(2000)
            item.also {
                println("아이템 발행: $item")
            }
        }
        .subscribe({ item ->
            Thread.sleep(2000)
            println("아이템 소비 : $item")
        }) { throwable ->
            throwable.printStackTrace()
        }
    Thread.sleep(30 * 1000L)
}
/**
 * 실행 결과
 * 아이템 발행: 0
 * 아이템 소비 : 0
 * io.reactivex.rxjava3.exceptions.MissingBackpressureException: Can't deliver value 128 due to lack of requests
 */
```

위와 같이 예외 사항이 발생할 수 있다는 것을 숙지하고, Flowable을 사용해야 합니다.

문제를 해결하도록 RxJava는 배압 제어 연산자를 제공 합니다.

## 배압 제어 연산자

이미 만들어진 Flowable에 대해 배압에 대한 전략을 설정할 수 있도록 도와주는 배압 제어 연산자를 적용할 수 있습니다.

#### **onBackpressureBuffer** 연산자
  - 배압이 구현 되지 않은 Flowable 에 대해 BackpressureStrategy.BUFFER 를 적용합니다.
  - 매개 변수별로 종류가 많으며 용량, 지연, 오버플로 콜백 등에 대한 것을 정의할 수 있습니다.

#### **onBackpressureLatest** 연산자
  - 스트림 버퍼가 가득 차면 최신의 아이템을 버퍼에 유지하려고 오래된 아이템을 버리는 연산자입니다.
  - **최신의 상태나 데이터만 의미가 있을 때 사용**하면 좋습니다.

#### **onBackpressureDrop** 연산자
  - 버퍼가 가득 찬 상태에서 버퍼에 든 아이템을 소비하는 쪽이 바쁘다면 발행된 아이템을 버립니다.

## Flowable 생성과 배압 전략

`Flowable.create()` 는 `Observable.create()` 과 비슷합니다.

EmitterBackpressure Stragtegy(배압 전략) 을 설정해야합니다.

배압 전략은 발행된 아이템들의 캐싱 및 폐기 여부를 지정하거나 아니면 배압 제어를 구현하지 않도록 설정할 수 있습니다.

```kotlin
@Test
fun test() {
    Flowable.create<Int>({ emitter ->
        (0..1000).forEach {
            //다운스트림 취소 및 폐기 시 true
            if (emitter.isCancelled) {
                return@create
            }
            emitter.onNext(it)
        }
        emitter.onComplete()
    }, BackpressureStrategy.BUFFER)
        .subscribeOn(Schedulers.computation())
        .observeOn(Schedulers.io())
        .subscribe(::println) { throwable ->
            throwable.printStackTrace()
        }
    Thread.sleep(100)
}
```

## 배압 전략

#### <span style="color:black">BackpressureStrategy.MISSING</span>

- 기본적으로 배압 전략을 구현하지 않는다. 오버플로를 다루려면 배압 제어 연산자를 사용하야 합니다.

#### <span style="color:black">BackpressureStrategy.ERROR</span>

- 스트림에서 소비자가 생산을 따라가지 못하는 경우 MissingBackpressureException 예외를 발생 시킵니다.

#### <span style="color:black">BackpressureStrategy.BUFFER</span>

- 구독자가 아이템을 소비할 때까지 발행한 아이템들을 버퍼에 넣어 둡니다.
- 해당 버퍼는 제한이 없는 큐지만, 가용 메모리를 벗어나는 경우 OOM(OutOfMemmoryError)를 발생시킬 수 있습니다.

#### <span style="color:black">BackpressureStrategy.DROP</span>

- 구독자가 아이템을 소비하느라 바빠서 생산자를 못 따라가는 경우 발행된 아이템을 모두 무시하고 버립니다.

#### <span style="color:black">BackpressureStrategy.LATEST</span>

- 구독자가 아이템을 받을 준비가 될 때까지 가장 최신의 발행된 아이템들만 유지하며 이전 아이템은 버립니다.




---

참고자료

[아키텍처를 알아야 앱 개발이 보인다](http://www.yes24.com/Product/Goods/89958199)