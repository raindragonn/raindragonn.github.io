---
layout: "post"
title: "[TIL] Android - RxJava (5) Subject"
subtitle: "ReactiveX"
date:       2021-08-16
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

## Subject

Subject는 Observable과 Observer를 모두 구현한 추상 타입으로 하나의 소스로부터 다중의 구독자에게 멀티 캐스팅이 가능하며, Observer를 구현하므로 `onNext()`, `onError()`, `onComplete()` 등의 메서드를 수동으로 호출하여 이벤트를 구독자들에게 전달할 수 있습니다.

#### <span style="color:black">PublishSubject</span>

- Subject를 구현한 가장 단순한 타입 중 한 가지로 구독자들에게 이벤트를 널리 전달합니다.

```kotlin
@Test
fun test() {
    PublishSubject.create<String>().apply {
        subscribe({item ->
            println("A: $item")
        }, {t ->
            t.printStackTrace()
        }, {
            println("A: onComplete()")
        })
        subscribe({item ->
            println("B: $item")
        }, {t ->
            t.printStackTrace()
        }, {
            println("B: onComplete()")
        })
        onNext("Hello")
        onNext("World")
        onNext("!!!")
        onComplete()
    }
    /**
     * 실행 결과
     * A: Hello
     * B: Hello
     * A: World
     * B: World
     * A: !!!
     * B: !!!
     * A: onComplete()
     * B: onComplete()
     */
}
```

`create()` 메서드를 통해 간단히 Subject 객체를 생성했습니다.

Subject는 Observable이자 Observer이므로 **발행과 구독을 모두 Subject객체를 통해 하는 것을 확인**할 수 있습니다.

><span style="color:black"> Subject를 이용할 때 주의해야 할 점은 Subject는 <span style="color:red">Hot Observable</span>이라는 사실을 잊지 말아야 합니다</span>

Subject는 Observer이기도 하므로, 다른 Observable의 구독자로 이벤트를 처리할 수도 있습니다.

소비하는 아이템은 다시 Observable로 발행하여 다른 구독자에게 전달합니다.

```kotlin
@Test
fun test() {
    val src1: Observable<Long> = Observable.interval(1, TimeUnit.SECONDS)
    val src2: Observable<Long> = Observable.interval(500, TimeUnit.MILLISECONDS)

    val subject: PublishSubject<String> = PublishSubject.create<String>()
    src1.map { item -> "A: $item" }.subscribe(subject)
    src2.map { item -> "B: $item" }.subscribe(subject)
    subject.subscribe(::println)
    Thread.sleep(5000)
}
/**
 * 실행 결과
 * B: 0
 * A: 0
 * B: 1
 * B: 2
 * A: 1
 * B: 3
 * B: 4
 * A: 2
 * B: 5
 * B: 6
 * A: 3
 * B: 7
 * B: 8
 * B: 9
 * A: 4
 */
```

Subject를 통해 구독하고 아이템을 재발행도 하고, `merge()` 연산자처럼 두 Observable소스를 묶어서 이벤트를 관리하는 것도 가능 하다는 것을 확인할 수 있습니다.

#### <span style="color:black"> SerializedSubject</span>

만약 서로 다른 스레드에서 Subject에 접근하여 아이템을 발행하는 상황에서는 Subject가 스레드 안전을 보장하지 않습니다.두 개의 스레드가 동시에 메모리에 접근하는 경우 아이템의 값을 보장할 수 없다 보니 연산자에서 잘못된 동작으로 Exception이 발생 할 수 있습니다.
이처럼 스레드에 안전하지 않은 경우에 `Subject.toSerialized()` 메서드를 통해 SerializedSubject를 객체로 생성할 수 있습니다.

- SerializedSubject는 접근 제어자가 public이 아니므로 RxJava 내부에서만 접근 가능한 타입입니다. 어플리케이션에서 이 타입에 접근은 불가능하지만 사용은 할 수 있습니다.

- **SerializedSubject는 내부에서 synchronized 키워드를 통해 스레드를 제어해 스레드에 안전한 Subject를 제공**합니다.

#### <span style="color:black"> BehaviorSubject</span>

- 기본적으로 PublishSubject와 동일하게 동작하지만, 새로운 Observer를 통해 구독 시 가장 마지막 아이템만을 발행 합니다.
- **가장 최근 상태값을 가져오는 것이 중요할 때 사용**할 수있습니다.

```kotlin
@Test
fun test() {
    val subject: BehaviorSubject<Int> = BehaviorSubject.create()

    subject.apply {
        subscribe{item -> println("A: $item") }
        onNext(1)
        onNext(2)
        subscribe{item -> println("B: $item") }
        onNext(3)
        subscribe{item -> println("C: $item") }
    }
}
/**
 * 실행 결과
 * A: 1
 * A: 2
 * B: 2
 * A: 3
 * B: 3
 * C: 3
 */
```

#### <span style="color:black"> ReplaySubject</span>

- PublishSubject에 `cache` 연산자를 적용한 것과 유사합니다.
- **새로운 구독자가 구독을 요청하면 이전에 발행했던 아이템 모두를 구독자에게 전달**합니다.
- 큰 볼륨을 갖는 아이템 발행 또는 무한한 아이템을 발행하는 소스에 대해서는 OOM(OutOfMemmoryException)을 염두해야 합니다.

```kotlin
@Test
fun test() {
    val subject: ReplaySubject<Int> = ReplaySubject.create()

    subject.apply {
        subscribe { item -> println("A: $item") }
        onNext(1)
        onNext(2)
        subscribe { item -> println("B: $item") }
        onNext(3)
        subscribe { item -> println("C: $item") }
    }
}
/**
 * 실행 결과
 * A: 1
 * A: 2
 * B: 1
 * B: 2
 * A: 3
 * B: 3
 * C: 1
 * C: 2
 * C: 3
 */
```

#### <span style="color:black">AsyncSubject</span>

- `onComplete()` 호출 직전에 발행된 아이템만을 구독자들에게 전달합니다.

```kotlin
@Test
fun test() {
    val subject: AsyncSubject<Int> = AsyncSubject.create()

    subject.apply {
        subscribe { item -> println("A: $item") }
        onNext(1)
        onNext(2)
        subscribe { item -> println("B: $item") }
        onNext(3)
        onComplete()
        subscribe { item -> println("C: $item") }
    }
}
/**
 * 실행 결과
 * A: 3
 * B: 3
 * C: 3
 */
```

#### <span style="color:black">UnicastSubject</span>

- Observer가 UnicastSubject에 구독하기 전까지는 발행하는 아이템을 버퍼에 저장하고, 구독이 시작될 때 버퍼에 있던 아이템을 모두 발행하고 버퍼를 비워 냅니다.
- 첫 번째 구독자가 모든 아이템을 다 소비해 두 번째 구독자부터는 아이템을 수신할 수 없기 때문에 **구독자를 여러 개 둘 수가 없습니다.**
- 두 번째 구독을 시도한다면 IllegalStateException 예외를 던집니다.

```kotlin
@Test
fun test() {
    val subject: UnicastSubject<Long> = UnicastSubject.create()
    Observable.interval(1,TimeUnit.SECONDS)
        .subscribe(subject)
    Thread.sleep(3000)
    subject.subscribe { item -> println("A: $item") }
    Thread.sleep(2000)
}
/**
 * 실행 결과
 * // ... 3초 가 흐른뒤
 * A: 0
 * A: 1
 * A: 2
 * A: 3
 * A: 4
 */
```

처음 3초동안은 발행된 아이템을 UnicastSubject의 버퍼에 축적해 콘솔에 아무런 출력이 없다가(3초 뒤에 구독하므로)3초 이후 축적된 아이템을 한번에 모두 방출하고 그 이후로 2초동안은 1초마다 발행된 아이템이 콘솔에 출력 됩니다.

---

참고자료

[아키텍처를 알아야 앱 개발이 보인다](http://www.yes24.com/Product/Goods/89958199)