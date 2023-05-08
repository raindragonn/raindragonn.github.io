---
layout: "post"
title: "[kotlin] 컬렉션 타입과 람다"
subtitle: "Collection, lambda"
date:       2021-05-01
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

## 컬렉션 타입과 람다

### 집합 연산 함수

#### forEach(), forEachIndexed()

|함수명|기능|
|---|---|
|`forEach()`|컬렉션 타입의 데이터 개수만큼 특정 구문을 반복 실행할 때 유용|
|`forEachIndexed()`|forEach() 함수와 동일하며, 람다 함수에 인덱스 값을 추가로 전달해 줍니다.|

ex)

```kotlin
val testList = (0..10).toList()

testList.forEach {
    println("result = $it")
}

testList.forEachIndexed { index, i ->
    println("index = $index, result = $i")
}
```

---

#### all(), any()

|함수명|기능|
|---|---|
|`all()`|특정 조건에 `모두 만족하는지 판단`할 때 사용합니다.|
|`any()`|특정 조건에 `만족하는 데이터가 있는지` 판단할 때 사용 합니다.|

ex)
```kotlin
val users = listOf(User("Lee", 27), User("Kim", 10))
users.all { it.age > 20 }
    .let {
        if(it)
            println("users 에는 age가 20이 넘는 사람 뿐이다.")
        else
            println("users 에는 age가 20이 넘지 않는 사람도 있다.")
    }

users.any { it.age > 20 }
    .let {
        if(it)
            println("users 중에 age가 20이 넘는 사람이 한명이라도 있다.")
        else
            println("users 중에 age가 20이 넘는 사람이 한명도 없다.")
    }
```

---

#### count(), find()

|함수명|기능|
|---|---|
|`count()`|람다식으로 대입한 `조건에 만족하는 데이터의 개수`를 반환 합니다.|
|`find()`|`조건에 만족하는 데이터중 가장 첫 번째 데이터`를 반환 합니다.(만족하는 데이터가 없으면 null)|


ex)
```kotlin
val users = listOf(User("Lee", 27), User("Kim", 10))
println("users에서 age가 20이 넘는 데이터의 수는 = ${users.count { it.age > 20 }}")

users.find { it.age > 20 }.let {
    println("users에서 age가 20이 넘는 첫 번째 데이터의 이름은 ${it?.name}")
}
```

---

#### reduce(), reduceRight(), fold(), foldRight()

|함수명|기능|
|---|---|
|`reduce()`|컬렉션 타입의 데이터를 람다 함수에 차례로 전달하며 람다 함수의결괏값을 그 `다음 데이터로 전달`해 줍니다.|
|`reduceRight()`|reduce()함수와 같지만 전달되는 데이터가 마지막부터 `거꾸로 전달` 합니다.|
|`fold()`|reduce() 동일하며 추가로 `초깃값을 지정할 수 있습니다.`|
|`foldRight()`|fold()함수와 같지만 전달되는 데이터가 마지막부터 거꾸로 전달되며 매개변수의 순서도 첫 번째가 전달되는 데이터, 두 번째가 이전 결괏값입니다.|


`fold(), foldRight() ex)`

```kotlin
listOf(1,2).fold(10, { total, next ->
    println("$total ... $next")
    total + next        // 람다식은 마지막이 반환값
}).let{
    println("fold test = $it")
}

listOf(1, 2).foldRight(10, { next, total ->
    println("$total ... $next")
    total + next
}).let {
    println("foldRight test = $it")
}
```

결과
```
10 ... 1
11 ... 2
fold test = 13
10 ... 2
12 ... 1
foldRight test = 13
```

`reduce(), reduceRight() ex)`
```kotlin
listOf(1,2,3,4,5,6,7,8,9,10).reduce { sum, next ->
    println("$sum ... $next")
    sum + next
}.let{
    println("전체의 합은 $it")
}

listOf(1,2,3,4,5,6,7,8,9,10).reduceRight{next, sum ->
    println("$sum ... $next")
    sum + next
}.let {
    println("전체 합은 $it")
}
```

결과
```
1 ... 2
3 ... 3
6 ... 4
10 ... 5
15 ... 6
21 ... 7
28 ... 8
36 ... 9
45 ... 10
전체의 합은 55
10 ... 9
19 ... 8
27 ... 7
34 ... 6
40 ... 5
45 ... 4
49 ... 3
52 ... 2
54 ... 1
전체 합은 55
```

---

#### max(), maxBy(), min(), minBy()

|함수명|기능|
|---|---|
|`max()`|컬렉션 타입의 데이터 중 `가장 큰 값을 반환`합니다.|
|`maxBy()`|목적은 max()와 같지만 `로직에 의한 결과를 기준으로` 가장 큰 값을 반환합니다.|
|`min()`|컬렉션 타입의 데이터 중 `가장 작은 값을 반환`합니다.|
|`minBy()`|목적은 min()와 같지만 `로직에 의한 결과를 기준으로` 가장 작은 값을 반환합니다.|


ex)
```kotlin
val max = listOf(1,11,5).max()
println("max test = $max")

val max2 = listOf(1,11,5).maxBy{ it % 5 }
println("maxBy text = $max2")

val min = listOf(1,11,5).min()
println("min test = $min")

val min2 = listOf(1,11,5).minBy{ it % 5 }
println("min text = $min2")

```

결과
```
max test = 11
maxBy text = 1
min test = 1
min text = 5
```

---

#### none(), sumBy()

|함수명|기능|
|---|---|
|`none()`|지정한 `조건에 맞는 데이터가 없으면 true, 있으면 false를 반환` 합니다|
|`sumBy()`|`람다 함수를 거쳐 반환한 모든 값`을 더하는 함수 입니다.|


ex)
```kotlin
val hasNotAdult = listOf(User("Lee",27),User("Kim",29)).none{ it.age > 20}
println("none test = $hasNotAdult")

val sumBy = listOf(1,2,3).sumBy { it * 2 }
println("sumBy test = $sumBy")
```

결과
```
none test = false
sumBy test = 12
```

---

### 필터링 함수

#### filter()

 - 컬렉션 타입의 데이터 중 `특정 조건에 맞는 데이터만 추출`할 때 이용합니다.

ex)
```kotlin
val list: List<Int> = listOf(10, 20, 5, 6, 30) 
val filterList: List<Int>= list.filter { it > 10 }
```

---

#### filterNot(), filterNotNull()

|함수명|기능|
|---|---|
|`filterNot()`|`조건에 맞지 않는 데이터`만 추출합니다.|
|`filterNotNull()`|`null이 아닌 데이터만` 추출합니다.|

---

#### drop(), dropWhile(), dropLastWhile()

|함수명|기능|
|---|---|
|`drop()`|매개변수의 `숫자 개수 만큼을 제외한 나머지를 추출합니다.`|
|`dropWhile()`|람다의 조건식이 false가 되는 순간 그 데이터부터 이후 데이터를 추출 합니다.|
|`dropLastWhile()`|`dropWhile()`함수와 순서가 반대라고 생각하면 됩니다.|

`특정 조건을 토대로 데이터를 제외한다고 생각하면 편합니다.`

ex)
```kotlin
listOf(1,2,3,4).drop(2).forEach {
    println(it)
}
// 3
// 4

listOf(2,1,12,5,23).dropWhile { it < 10 }
    .forEach {
        println(it)
    }
// 10 보다 작은 조건을 만족하는 2,1을 drop 한다고 생각하면 편합니다.

// 12
// 5
// 23
```

---

#### slice(), take(), takeLast(), takeWhile()

 - 특정 위치에 있는 데이터를 추출하는 함수입니다.

|함수명|기능|
|---|---|
|`slice()`|범위나 숫자 값을 여러개 대입하고 `그 위치에 있는 데이터만` 추출합니다.|
|`take()`|맨 `처음 데이터부터 매개변수의 값 만큼` 데이터를 추출합니다.|
|`takeLast()`|take함수와 반대로 `마지막 데이터부터 매개변수의 값 만큼` 데이터를 추출합니다.|
|`takeWhile()`|람다 함수로 조건을 명시하고 `조건에 맞지 않는 데이터가 나오기 전까지의 데이터`를 추출합니다.|

ex)
```kotlin
listOf(1, 11, 5, 25, 3, 2).slice(1..3)
    .forEach { println("slice : $it") }

listOf(1, 11, 5, 25, 3, 2).slice(listOf(2,3))
    .forEach { println("slice : $it ") }

listOf(1, 11, 5, 25, 3, 2).take(3)
    .forEach { println("take : $it") }

listOf(1, 11, 5, 25, 3, 2).takeLast(3)
    .forEach { println("takeLast : $it") }

listOf(1, 11, 5, 25, 3, 2).takeWhile { it < 20 }
    .forEach { println("takeLastWhile : $it") }
```

결과

```
slice : 11
slice : 5
slice : 25
slice : 5 
slice : 25 
take : 1
take : 11
take : 5
takeLast : 25
takeLast : 3
takeLast : 2
takeLastWhile : 1
takeLastWhile : 11
takeLastWhile : 5
```

---

### 매핑 함수

#### map(), mapIndexed()

|함수명|기능|
|---|---|
|`map()`|forEach()함수와 비슷하게 데이터 개수만큼 반복 실행하며 결과를 반환합니다.|
|`mapIndexed()`|`map()`함수와 동일하지만 람다 함수에 인덱스까지 전달합니다.|


ex)
```kotlin
listOf(1, 11, 5).map { it * 10 }
    .forEach { println(it) }

listOf(1, 11, 5).mapIndexed { index, i -> i * index }
    .forEach { println(it) }
```

결과

```
10
110
50
0
11
10
```

---

#### groupBy()

 - 컬렉션 타임의 데이터를 특정 데이터로 묶을 때 사용합니다.

 - `Map타입으로 반환합니다.`
 
 - `람다식의 반환값을 기준으로 다른경우 map의 키로, value에는 해당 키에 맞는 data가 모인 list가 반환 됩니다.`

```kotlin
public inline fun <T, K> Iterable<T>.groupBy(keySelector: (T) -> K): Map<K, List<T>> 
```

```kotlin
data class User(
    val name: String,
    val age: Int
)

listOf(
    User("Lee", 10),
    User("Kim", 24),
    User("Lee", 42),
    User("Kim", 13)
).groupBy { it.name }
    .forEach {
        println("it.key = ${it.key} it.value = ${it.value.count()}")
        it.value.forEach { user ->
            println("${user.name} , ${user.age}")
        }
    }
```

결과

```
it.key = Lee it.value = 2
Lee , 10
Lee , 42
it.key = Kim it.value = 2
Kim , 24
Kim , 13
```

---

### 요소 함수

#### contains()

 - 컬렉션 타입의 데이터 중 `특정 데이터가 있는지를 판단하는 함수`입니다.

 - 데이터가 있으면 true, 없으면 false를 반환합니다.

ex)
```kotlin
listOf(2,5,10,6).contains(7)
    .let {
        println("containsTest = $it")
    }

// containsTest = false
```

---

#### first(), firstOfNull(), last(), lastOrNull()

|함수명|기능|
|---|---|
|`first()`|람다식의 `조건으로 맞는 가장 첫 번째 데이터`를 추출합니다.조건에 맞는 데이터가 없으면 NoSuchElementException이 발생|
|`firstOfNull()`|first()와 동일하지만 조건에 맞는 데이터가 없으면 null을 반환합니다.|
|`last()`|람다식의 `조건으로 맞는 가장 마지막 데이터`를 추출합니다.다.조건에 맞는 데이터가 없으면 NoSuchElementException이 발생|
|`lastOrNull()`|last()와 동일하지만 조건에 맞는 데이터가 없으면 null을 반환합니다.|

ex)

```kotlin
println("firstTest = ${listOf(2, 5, 10, 8).first { it % 2 == 0 }}")
println("lastTest = ${listOf(2, 5, 10, 8).last { it % 2 == 0 }}")
println("firstOrNullTest = ${listOf(2, 5, 10, 8).firstOrNull { it % 2 == 0 }}")
println("lastOrNullTest = ${listOf(2, 5, 10, 8).lastOrNull { it % 2 == 0 }}")
```

결과

```
firstTest = 2
lastTest = 8
firstOrNullTest = 2
lastOrNullTest = 8
```



---

참조 : 깡샘의 코틀린 프로그래밍