---
layout: "post"
title: "[TIL] Android - RxJava (3) 스케줄러"
subtitle: "ReactiveX"
date:       2021-08-12
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
    - RxJava
    - ReactiveX
---

## 스케줄러

RxJava에서는 스케줄러(Scheduler)를 통해 멀티 스레드와 같은 비동기 작업을 돕습니다. RxJava에서는 Schedulers 클래스에서 제공하는 정적 팩토리 메서드를 통해 스케줄러를 설정할 수 있습니다.

```kotlin
val ioScheduler : Scheduler = Schedulers.io()
val computationScheduler : Scheduler = Schedulers.computation()
val trampolineScheduler : Scheduler = Schedulers.trampoline()
val newScheduler : Scheduler = Schedulers.newThread()
//for Android
val mainScheduler : Scheduler = Schedulers.mainThread()
```

### 스케줄러의 종류

#### <span style="color:black">IO 스케줄러</span>

- 네트워크, 데이터베이스, 파일 시스템 환경 등의 **블로킹 이슈가 발생하는 곳에서 비동기적인 작업을 위해 사용**될 수 있습니다.

#### <span style="color:black">newThread 스케줄러</span>

- **매번 새로운 스케줄러(스레드)**를 생성합니다.

#### <span style="color:black">Computation 스케줄러</span>

- 단순 반복적인 작업, 콜백 처리 그리고 다른 계산적인 작업에 사용됩니다. **블로킹 이슈가 발생하는 곳에서 사용하는 것을 추천하지 않습니다.**

#### <span style="color:black">Trampoline 스케줄러</span>

- 새로운 스레드를 생성하지 않고 현재 스레드에 무한한 크기의 큐를 생성하는 스케줄러 입니다. **모든 작업을 순차적으로 실행하는 것을 보장** 합니다.

### `subscribeOn` 과 `observeOn`

RxJava에서는 `subscribeOn`과 `observeOn` 연산자를 통해 스케줄러를 이용하며, 이를통해 **간단히 멀티 스레딩을 구현**할 수 있습니다.

기본적으로 Observer가 선언되고 구독되는 스레드에서 동작 합니다.(안드로이드의 경우 메인스레드(UI 스레드))

```kotlin
@Test
fun test() {
    val src: Observable<Int> = Observable.create { emitter ->
        (0..2).forEach {
            val threadName = Thread.currentThread().name
            println("#Subs on $threadName : $it")
            emitter.onNext(it)
            Thread.sleep(100)
        }
        emitter.onComplete()
    }
    src.subscribe {s ->
        val threadName = Thread.currentThread().name
        println("#Obsv on $threadName : $s")
    }
}
/**
 * 실행 결과
 * #Subs on main : 0
 * #Obsv on main : 0
 * #Subs on main : 1
 * #Obsv on main : 1
 * #Subs on main : 2
 * #Obsv on main : 2
 */
```

스레드 이름을 출력해 보면 모두 메인 스레드에서 동작하는 것을 확인할 수 있습니다.

#### `subscribeOn` 연산자

`subscribeOn` 연산자를 사용하여 스레드를 지정 한 경우는 아래와 같습니다.

```kotlin
@Test
fun test() {
    val src: Observable<Int> = Observable.create { emitter ->
        (0..2).forEach {
            val threadName = Thread.currentThread().name
            println("#Subs on $threadName : $it")
            emitter.onNext(it)
            Thread.sleep(100)
        }
        emitter.onComplete()
    }

    src.subscribeOn(Schedulers.io())
        .subscribe { s ->
            val threadName = Thread.currentThread().name
            println("#Obsv on $threadName : $s")
        }
    Thread.sleep(500)
}
/**
 * 실행 결과
 * #Subs on RxCachedThreadScheduler-1 : 0
 * #Obsv on RxCachedThreadScheduler-1 : 0
 * #Subs on RxCachedThreadScheduler-1 : 1
 * #Obsv on RxCachedThreadScheduler-1 : 1
 * #Subs on RxCachedThreadScheduler-1 : 2
 * #Obsv on RxCachedThreadScheduler-1 : 2
 */
```

`subscribeOn`연산자는 Observable 소스에 **어떤 스케줄러를 사용하여 아이템을 발행할지 알려 줍니다.**

`subscribeOn` 연산자만 있고 `ObserveOn` 연산자가 없다면 해당 스케줄러는 아이템 발행 및 구독 까지 작용합니다.

#### `observeOn` 연산자

아래는 `observeOn` 연산자를 사용한 예시 입니다.

```kotlin
@Test
fun test() {
    val src: Observable<Int> = Observable.create { emitter ->
        (0..2).forEach {
            val threadName = Thread.currentThread().name
            println("#Subs on $threadName : $it")
            emitter.onNext(it)
            Thread.sleep(100)
        }
        emitter.onComplete()
    }

    src.subscribeOn(Schedulers.io())
        .observeOn(Schedulers.computation())
        .subscribe { s ->
            val threadName = Thread.currentThread().name
            println("#Obsv on $threadName : $s")
        }
    Thread.sleep(500)
}
/**
 * 실행 결과
 * #Subs on RxCachedThreadScheduler-1 : 0
 * #Obsv on RxComputationThreadPool-1 : 0
 * #Subs on RxCachedThreadScheduler-1 : 1
 * #Obsv on RxComputationThreadPool-1 : 1
 * #Subs on RxCachedThreadScheduler-1 : 2
 * #Obsv on RxComputationThreadPool-1 : 2
 */
```

`observeOn` 연산자를 이용하면 Observable에서 발행된 아이템을 가로채어 **해당 스케줄러로 아이템을 구독**합니다.

아이템 발행시(`subscribeOn`)와 구독 시(`observeOn`)에 스레드의 이름이 다른것을 확인할 수 있습니다.

> `interval, timer, replay, buffer` 등의 연산자는 computation 스케줄러로 고정되어 다른 스케줄러를 지정하더라도 무시 됩니다.


---

참고자료

[아키텍처를 알아야 앱 개발이 보인다](http://www.yes24.com/Product/Goods/89958199)