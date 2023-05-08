---
layout: "post"
title: "[TIL] Android - RxJava (1)"
subtitle: "ReactiveX"
date:       2021-08-09
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

# RxJava란?

- [ReactiveX(Reactive Extensions)][reactiveXIo]를 자바로 구현한 라이브러리
- 이벤트 처리 및 비동기 처리의 구성에 최적화된 라이브러리
- Observable 추상화 및 관련 함수에 중점을 둔 단일 JAR로 가벼운 라이브러리

## 왜 RxJava를 배워야 할까?

안드로이드는 사용자의 이벤트에 따라 즉각적으로 반응하는 것이 중요하고

동시성문제, 다중 이벤트 처리, 백그라운드 스레드 처리 등에서 생길 수 있는 많은 문제점에 대해 범용적이고 확실한 해결책을 제시합니다.

기존에 작성한 비즈니스 로직에 새로운 프로세스가 추가된다고 하더라도 연산자 단위로 처리하므로

어떠한 기능을 추가하거나 제거하는 것이 매우 간단합니다.

## 반응형 프로그래밍?

반응형 프로그래밍(Reactive Programming)이란 **주변 환경과 끊임없이 상호 작용을 하는 프로그래밍**을 의미

프로그램이 주도하는 것이 아닌 환경이 변하면 이벤트를 받아 동작하도록 만드는 프로그래밍 기법 입니다.

### 명령형 프로그래밍과 반응형 프로그래밍의 차이

#### <span style="color:black">1.명령형 프로그래밍</span>

명령형 프로그래밍은 작성된 코드가 정해진 순서대로 실행되는 방식으로, 

**어떻게(How to)**에 초점이 맞춰진 프로그래밍 기법 입니다.

```kotlin
@Test
fun test() {
    val items = arrayListOf<Int>()
    items.add(1)
    items.add(2)
    items.add(3)
    items.add(4)
    for(item in items){
        if(item % 2 == 0){
            println(item)
        }
    }
    items.add(5)
    items.add(6)
    items.add(7)
    items.add(8)
}
```

위의 코드는 다음과 같이 정리할 수 있습니다.
1. 리스트를 만든다.
2. 리스트에 1부터 4까지 순차적으로 아이템을 추가
3. items라는 리스트를 순회하며, 짝수를 출력
4. 리스트에 5부터 8까지 아이템을 순차적으로 추가

#### <span style="color:black">2.반응형 프로그래밍</span>

비동기적인 데이터의 흐름을 관찰하고 처리하는 방법 방식입니다.

**무엇(What)**에 초점이 맞춰진 프로그래밍 기법 입니다.

```kotlin
@Test
fun test2() {
    val items = PublishSubject.create<Int>()
    items.onNext(1)
    items.onNext(2)
    items.onNext(3)
    items.onNext(4)

    items.filter {
        it % 2 == 0
    }.subscribe(
        ::println
    )
    items.onNext(5)
    items.onNext(6)
    items.onNext(7)
    items.onNext(8)
}
```
위의 코드는 다음과 같이 정리할 수 있다.
1. 데이터 스트림을 만든다.(PublishSubject = 구독 시점 이후의 데이터만 옵저버에 전달)
2. 데이터 스트림에 1부터 4까지 순차적으로 추가
3. 데이터 스트림에서 짝수만 출력하는 데이터 스트림으로 변형한 뒤 구독
4. 데이터 스트림에 5부터 8까지 순차적으로 추가


## 1. Observable

RxJava에서는 Observable을 구독(subscribe)하는 Observer가 존재하고, Observable이 순차적으로 발행하는 데이터에 대해서 반응 

Observable은 다음 3가지 이벤트를 사용하여 동작 합니다.

1. `onNext()` : 하나의 소스 Observable에서 Observer까지 한 번에 하나씩 순차적으로 **데이터를 발행**
2. `onComplete()` : **데이터 발행이 끝났음**을 알리는 완료 이벤트를 전달, Observer에게 더는 `onNext()`가 발생하지 않음을 나타 냅니다.
3. `onError()` : **오류가 발생했음**을 Observer에게 전달 합니다.

위의 세 가지 메서드 들은 Emitter라는 인터페이스에 선언되며, Null을 발핼할 수 없음

### 1.1. Observable 생성하기

RxJava에서는 연산자(Operator)라고 부르는 여러 정적 메서드를 통해 기존 데이터를 참조하거나 변형하여 Observable을 생성할 수도 있습니다.

#### `create()` 연산자

`Observable.create()`를 사용하면 Emitter를 이용하여 직접 아이템을 발행 및 발행완료, 오류의 알림을 직점 설정할 수 있습니다.

```kotlin
@Test
fun test() {
    val source = Observable.create<String> {emitter ->
        emitter.onNext("Hello")
        emitter.onNext("World")
        emitter.onComplete()
    }
    source.subscribe(::println)
}
/**
 * 실행 결과
 * Hello
 * World
 */
```

Emitter를 통해 문자열 "Hello"와 "World"를 발행, `subscribe()`를 통해 구독하고 아이템의 발행이 끝났다면 반드시
`onComplete()`를 호출해야 합니다. 해당 함수 호출 후에 발행하는 데이터는 구독자가 데이터를 통지받지 못합니다.

```kotlin
@Test
fun test() {
    val source = Observable.create<String> {emitter ->
        emitter.onNext("Hello")
        emitter.onComplete()
        emitter.onNext("World")
    }
    source.subscribe(::println)
}
/**
 * 실행 결과
 * Hello
 */
```

오류가 발생했을 경우에는 Emitter를 통해 `onError()`를 호출해야 하며 구독자는 이를 적절히 처리해야 합니다.

```kotlin
@Test
fun test() {
    val source = Observable.create<String> { emitter ->
        emitter.onNext("Hello")
        emitter.onError(Throwable("Error!!"))
        emitter.onNext("World")
    }
    source.subscribe(::println) {throwable ->
        throwable.printStackTrace()
    }
}
/**
 * 실행 결과
 * Hello
 * java.lang.Throwable: Error!!
 * ....
 */
```

`create()` 연산자는 개발자가 직접 Emitter를 제어하므로 주의하여 사용해야 합니다.

예를들어 Observable이 폐기되었을 때 등록된 콜백을 모두 해제하지 않으면 메모리 누수가 발생하고, BackPressure(배압)를 직접 처리해야 합니다.

#### `Just()` 연산자

`Observable.just()` 연산자는 해당 아이템을 그대로 발행하는 Observable을 생성해 줍니다.
인자로 넣은 아이템을 차례로 발행, 한 개의 아이템을 넣을 수도 있으며, 타입이 같은 여러 개의 아이템을 넣을 수도 있습니다.

```kotlin
@Test
fun test() {
    val source = Observable.just("Hello", "World")
    source.subscribe(::println)
}
/**
 * 실행 결과
 * Hello
 * World
 */
```

만약 아무런 아이템을 발행하지 않는 빈 Observable을 만들고 싶다면, `Observable.empty()`연산자를 사용합니다.

### 1.2. 간단히 Obserable로 변환하기

이미 참조할 수 있는 배열 및 리스트 등의 자료구조나 Future, Callable 또는 Publisher 가 있다면 from으로 시작하는 연산자를 통해 Observable로 변환할 수 있습니다.

| 연산자 이름     | 설명                                                         |
| --------------- | ------------------------------------------------------------ |
| `fromArray()`     | 배열을 ObservableSource로 변환하여 아이템을 순차적으로 발행  |
| `fromIterable()`  | ArrayList, HashSet처럼 Iteravle을 구현한 모든 객체를 ObservableSource로 변환하여 아이템을 순차적으로 발행 |
| `fromFuture()`    | Future 인터 페이스를 지원하는 모든 객체를 ObservableSource로 변환하고 Future.get() 메서드를 호출한 값을 반환 |
| `fromPublisher()` | Publisher를 Observable 로 변환                               |
| `fromCallable()`  | Callable을 Observable 로 변환                                |


#### `fromArray()` 연산자

가지고 있는 아이템들이 배열일 경우에는 `fromArray()` 연산자를 이용하여 아이템을 순차적으로 발행할 수 있습니다.

```kotlin
@Test
fun test() {
    val itemArray = arrayOf("A","B","C")
    val source: Observable<String> = Observable.fromArray(*itemArray)
    source.subscribe(::println)
}
/**
 * 실행 결과
 * A
 * B
 * C
 */
```

#### `fromIterable()` 연산자

ArrayList, HashSet 등과 같이 일반적으로 Iterable을 구현한 자료 구조 클래스는 `fromIterable()`연산자를 통해 쉽게 Observable로 변환이 가능합니다.

```kotlin
@Test
fun test() {
    val itemArray = arrayListOf<String>("A","B","C")
    val source: Observable<String> = Observable.fromIterable(itemArray)
    source.subscribe(::println)
}
/**
 * 실행 결과
 * A
 * B
 * C
 */
```

#### `fromFuture()` 연산자

Future 인터페이스는 비동기적인 작업의 결과를 구할 때 사용 합니다.
보통 Executor Service를 통해 비동기적인 작업을 할 때 사용하며, Future 또한 `fromFuture()` 연산자를 통해 Observable로 변경이 가능.
Emitter 는 Observable 내부에서 `Futrue.get()` 메서드를 호출하고, Future의 작업이 끝나기 전까지 스레드는 블로킹 됩니다.

```kotlin
@Test
fun test() {
    val future = Executors.newSingleThreadExecutor().submit<String> {
        Thread.sleep(5000)
        "Hello World"
    }
    val source = Observable.fromFuture(future)
    source.subscribe(System.out::println)

}
/**
 * 실행 결과
 * (약 5초 뒤에)
 * Hello World
 */
```

RxJava에서는 Executor를 직접 다루기보다는 RxJava에서 제공하는 스케줄러를 사용하는것을 권장 합니다.

#### `fromPublisher()` 연산자

Publisher는 잠재적인 아이템 발행을 제공하는 생산자로 Subscriber 로부터 요청을 받아 아이템을 발행 합니다.

`fromPublisher()` 연산자를 통해 Publisher를 Observable로 다음과 같이 변환 가능

```kotlin
@Test
fun test() {
    val publisher = Publisher<String> {subscriber ->
        subscriber.onNext("A")
        subscriber.onNext("B")
        subscriber.onNext("C")
        subscriber.onComplete()
    }
    val source = Observable.fromPublisher<String>(publisher)
    source.subscribe(System.out::println)

}
/**
 * 실행 결과
 * A
 * B
 * C
 */
```


#### `fromCallable()` 연산자

Runnable과는 다르게 Callable 인터페이스는 비동기적인 실행 결과를 반환 합니다.

`fromCallable()` 연산자를 통해 Callable을 Observable로 변환하고, 비동기적으로 아이템을 발행할 수 있습니다.

```kotlin
@Test
fun test() {
    val callBack = Callable {
        "Hello World"
    }
    val source = Observable.fromCallable(callBack)
    source.subscribe(System.out::println)

}
/**
 * 실행 결과
 * Hello World
 */
```

### 1.3. 다양한 Observable의 형태

Observable 이외에 조금은 특별한 스트림들이 존재합니다.

1. Single
2. Maybe
3. Completable


#### `Single`

단 하나의 아이템만을 발행하는 특징이 있습니다.

그러므로 `just()` 연산자에는 **하나의 인자만**을 취할 수 있습니다.

Single은 단일 아이템을 발행한다는 점에서 **HTTP 요청/응답과 같은 이벤트를 처리하는 경우 자주 사용됩니다.**

```kotlin
@Test
fun test() {
    Single.just("Just Hello World")
        .subscribe(::println)
}
/**
 * 실행 결과
 * Just Hello World
 */
```

`create()` 연산자를 사용하는경우 Emitter를 사용하여 데이터를 발행합니다.

데이터를 한 번만 발행하므로 `OnNext()`와 `onComplete()` 메서드를 호출하는 대신 `onSuccess(T)`로 두 메서드를 반 헌에 대체 합니다.

```kotlin
@Test
fun test() {
    Single.create<String> { emitter ->
        emitter.onSuccess("Single Hello World")   
    }.subscribe(::println)
}
/**
 * 실행 결과
 * Single Hello World
 */
```

오류를 다루는 경우 Observable의 Emitter와 동일하게 `onError()`를 호출하여 오류를 구독자에게 통지할 수 있다.

몇 가지 RxJava의 연산자들은 Observable을 Single로 변환시키곤 합니다.

```kotlin
@Test
fun test() {
    val src = Observable.just(1, 2, 3)

    val singleSrc1: Single<Boolean> = src.all { it > 0 }
    val singleSrc2: Single<Int> = src.first(-1)
    val singleSrc3: Single<List<Int>> = src.toList()
}
```

또한 Single도 필요에 따라 Observable로 변환해야 하는 경우가 있습니다.

그런 경우 `toObservable()`연산자를 사용할 수 있습니다.

```kotlin
@Test
fun test() {
    val singleSrc : Single<String> = Single.just("Just Hello World")
    val observableSrc : Observable<String> = singleSrc.toObservable()
}
```

모든 소스의 경우 Observable로 변환하는 것뿐만 아니라 `to~` 연산자를 통해 다른 소스 형태로 바꾸는 것이 가능 합니다.

#### `Maybe`

Maybe는 아이템을 발행하거나 발행하지 않을 수도 있습니다.

아이템을 발행할 때는 `onSuccess()` 를 호출하고, 발행하지 않을 때는 `onComplete()`를 호출합니다.

`onSuccess()` 를 호출하는 경우 `onComplete()`를 호출할 필요는 없습니다.

```kotlin
@Test
fun test() {
    Maybe.create<Int> { emitter ->
        emitter.onSuccess(100)
        emitter.onComplete()    // 무시됨
    }
        .doOnSuccess { println("doOnSuccess") }
        .doOnComplete { println("doOnComplete") }
        .subscribe(::println)

    Maybe.create<Int> { emitter ->
        emitter.onComplete()
    }
        .doOnSuccess { println("doOnSuccess") }
        .doOnComplete { println("doOnComplete") }
        .subscribe(::println)
}
/**
 * 실행 결과
 * doOnSuccess
 * 100
 * doOnComplete
 */    
```

몇 가지 Observable 연산자는 반환 타입을 Maybe로 변환합니다.

```kotlin
@Test
fun test() {
    val observable: Observable<Int> = Observable.just(1, 2, 3)
    val maybe: Maybe<Int> = observable.firstElement()
    maybe.subscribe(::println)

    val observable2: Observable<Int> = Observable.empty()
    val maybe2: Maybe<Int> = observable2.firstElement()
    maybe2.subscribe(
        ::println,
        { throwable ->
            throwable.printStackTrace()
        }, {
            println("onComplete")
        }
    )
}
/**
 * 실행 결과
 * 1
 * onComplete
 */
```

#### `Completable`

Completable은 아이템을 발행하지 않으며, `onComplete()`와 `onError()`만 존재 합니다.

```kotlin
@Test
fun test() {
    Completable.create { emitter ->
        // do Something here
        emitter.onComplete()
    }.subscribe {
        println("completed1")
    }

    Completable.fromRunnable {
        //do Something here
    }.subscribe{
        println("completed2")
    }
}
/**
 * 실행 결과
 * completed1
 * completed2
 */
```

### 1.4. Cold Observable과 Hot Observable의 차이


#### <span style="color:black">Cold Observable</span>

구독을 요청하면 아이템을 발행하기 시작합니다.

아이템은 처음부터 끝까지 발행되며, 임의로 종료시키지 않는 이상 **여러 번 요청에도 처음부터 끝까지 발행하는 것을 보장** 합니다.

```kotlin
@Test
fun test() {
    val src = Observable.interval(1, TimeUnit.SECONDS)
    src.subscribe { value ->
        println("#1: $value")
    }
    Thread.sleep(3 * 1000)

    src.subscribe{value ->
        println("#2: $value")
    }
    Thread.sleep(3 * 1000)

}
/**
 * 실행 결과
 * #1: 0
 * #1: 1
 * #1: 2
 * #1: 3
 * #2: 0
 * #2: 1
 * #1: 4
 * #2: 2
 * #1: 5
 */
```

Observable을 구독하고 3초 뒤에 새로운 구독자로 다시 구독했을 때도 처음부터 다시 아이템을 발행하는 것을 볼 수 있습니다.

#### <span style="color:black">Hot Observable</span>

Hot Observable은 아이템 발행이 시작된 이후로 **모든 구독자에게 동시에 같은 아이템을 발행**합니다.

두 개의 구독자가 같은 Observable을 구독한다고 가정 했을 때 이 둘은 같은 아이템을 동시에 수신하지만, 두번째 구독자는 구독하기 전에 발행된 아이템을 놓칠 수도 있습니다.


#### `publish()` 연산자와 `connect()` 연산자

`ConncectableObservable`은 Hot Observable을 구현할 수 있도록 도와주는 타입으로 아무 Observable 타입이나 `publish()` 연산자를 이용하여 간단히 ConnectableObservable로 변환할 수 있습니다.

해당 타입은 구독을 요청해도 Observable은 데이터를 발행하지 않으며, `connect()` 연산자를 호출할 때 비로소 아이템을 발행하기 시작합니다.

```kotlin
@Test
fun test() {
    val src: ConnectableObservable<Long> = Observable.interval(1, TimeUnit.SECONDS).publish()
    src.connect()
    src.subscribe { value ->
        println("#1 : $value")
    }
    Thread.sleep(3 * 1000)
    src.subscribe { value ->
        println("#2 : $value")
    }
    Thread.sleep(3 * 1000)
}
/**
 * 실행 결과
 * #1 : 0
 * #1 : 1
 * #1 : 2
 * #1 : 3
 * #2 : 3
 * #1 : 4
 * #2 : 4
 * #1 : 5
 * #2 : 5
 */
```

#### `autoConnect()` 연산자

`autoConnect()` 연산자는 `connect()` 연산자를 호출하지 않더라도, 구독 시에 즉각 아이템을 발행할 수 있도록 도와주는 연산자 입니다.

`autoConnect()` 연산자의 매개변수는 아이템을 발행하는 구독자 수로, `autoConnect(2)`일 경우 구독자가 2개 이상 붙어야 아이템을 발행하기 시작 합니다.

```kotlin
@Test
fun test() {
    val src: Observable<Long> = Observable.interval(100,TimeUnit.MILLISECONDS)
        .publish()
        .autoConnect(2)

    src.subscribe{value ->
        println("A : $value")
    }
    src.subscribe{value ->
        println("B : $value")
    }
    Thread.sleep(500)
}
/**
 * 실행 결과
 * A : 0
 * B : 0
 * A : 1
 * B : 1
 * A : 2
 * B : 2
 * A : 3
 * B : 3
 * A : 4
 * B : 4
 */
```

만약 `src.subscribe()`를 한 번만 호출했다면 아이템을 발행하지 않아 아무것도 출력되지 않습니다.

`autoConnect()`의 매개 변수로 0 이하를 입력할 경우 구독자 수와 관계 없이 곧바로 아이템 발행을 시작합니다.

매개 변수를 지정하지 않을경우 `autoConnect(1)`과 동일하게 동작하며, 구독하자마자 아이템 발행을 시작합니다.

#### <span style="color:black">Disposable 다루기</span>

`subscribe()`메서드를 호출하면 Disposable 객체를 반환 합니다.

유한한 아이템을 발행하는 Observable의 경우 `onComplete()` 호출로 안전하게 종료 됩니다.

하지만 무한하게 아이템을 발행하거나 오랫동안 실행되는 Observable의 경우 이미 활성화 된경우 구독이 더는 필요하지 않을 수 있습니다.

이럴 때는 메모리 누수 방지를 위해 명시적인 폐기(dispose)가 필요합니다.

아래 예제는 무한히 아이템을 발행하는 Observable을 중간에 중지하는 예시 입니다.

```kotlin
@Test
fun test() {
    val disposable: Disposable = Observable.interval(1, TimeUnit.SECONDS)
        .subscribe(::println)

    Runnable {
        try {
            Thread.sleep(3500)
        } catch (e: Exception) {
            e.printStackTrace()
        }
        disposable.dispose()
    }.run()

}
/**
 * 실행 결과
 * 0
 * 1
 * 2
 */
```

`Disposable.dispose()`를 통해 아이템의 발행을 중지하고 모든 리소스를 폐기 할 수 있습니다.

`Disposable.isDisposed()`메서드를 통해 리소스가 폐기되었는지 확인할 수 있으며, `dispose()` 메서드 내부에서 폐기 여부를 체크하므로 `isDisposed()`의 결과를 확인하고 `dispose()` 메서드를 호출할 필요는 없습니다. 

또한, `onComplete()`를 명시적으로 호출하거나 호출됨을 보장한다면 `dispose()`를 호출할 필요 없습니다.


#### <span style="color:black">CompositeDisposable</span>

구독자가 여러 곳에 있으며, 이들을 폐기하려면 각각의 Disposabe 객체에 대해서 `dispose()`를 호출해야하지만, CompositeDisposable을 통해 한꺼번에 폐기할 수 있습니다.

```kotlin
@Test
fun test() {
    val source = Observable.interval(1,TimeUnit.SECONDS)
    
    val d1 : Disposable = source.subscribe(::println)
    val d2 : Disposable = source.subscribe(::println)
    val d3 : Disposable = source.subscribe(::println)
    val compositeDisposable : CompositeDisposable = CompositeDisposable().apply {
        add(d1)
        add(d2)
        add(d3)
        //또는
        addAll(d1,d2,d3)
    }
    // 특정시점에 폐기하기
    compositeDisposable.dispose()
}
```




---

참고자료

[아키텍처를 알아야 앱 개발이 보인다](http://www.yes24.com/Product/Goods/89958199)

[RxJava 가 뭐야? 새로운 언어야?](https://velog.io/@haero_kim/Android-대체-사람들이-RxJava-거리는-게-뭐야-Reactive-X-단-번에-이해하기)



[reactiveXIo]:http://reactivex.io/