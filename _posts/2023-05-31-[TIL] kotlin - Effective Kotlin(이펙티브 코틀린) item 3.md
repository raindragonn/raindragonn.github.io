---
layout: "post"
title: "[TIL] Kotlin - 최대한 플랫폼 타입을 사용하지 말라"
subtitle: "Effective kotlin - Item 3"
date: 2023-05-31
author: "raindragonn"
lang: ko
categories: [TIL - Kotlin]
tags:
  - Kotlin
  - Effective Kotlin
  - 이펙티브 코틀린
---

## 최대한 플랫폼 타입을 사용하지 말라

## 들어가기에 앞서..

코틀린의 주요기능으로는 **널 안정성(null-safety)**이 있다. 자바에서 자주 만나는 NPE를 코틀린에서는 거의 찾아보기가 힘들다. null-safety 메커니즘이 없는 자바, C등의 언어를 코틀린과 연결해서 사용하느 경우에는 이러한 예외가 발생할 수 있다.

## Java에서는 어떨까?

- `@Nullable` 어노테이션이 붙은 경우 이를 nullable로 추정해서 Type?로 변경하면된다.
- `@NotNull` 어노테이션이 붙은 경우 Type으로 변경하면된다.
- 위와 같은 어노테이션이 없는 경우에는 nullable로 가정하고 다루어야한다.

## 코틀린에서 다른 언어에서 넘어온 타입을 다루는 방법

- 이러한 타입을 **플랫폼 타입**이라고 부른다.
    - 다른 프로그래밍 언어에서 전달되어서 nullable인지 아닌지 알 수 없는 타입을 말한다.

### 플랫폼 타입을 사용하는 방법

- 플랫폼 타입은 String! 처럼 타입 뒤에 ! 기호를 붙여서 표기한다.

```java
// java
public class UserRepo {
		public User getUser() {
				/// ... 
		}
}
```

```kotlin
/// kotlin
val repo = UserRepo()
val user1 = repo.user // user1의 타입은 User!
val user2: User = repo.user //user2의 타입은 User
val user3: User? = repo.user // user3의 타입은 User?
```

- 위 User2의 경우 null이 아니라고 생각하지만 null일 가능성이 있어 위험하다.
    - 언제든 어노테이션으로 표시하거나 주석으로 달아두지 않는경우 언제든지 동작이 변경될 가능성이 있다.

<aside>
💡 자바를 코틀린과 사용하는 경우 가능한 `@Nullable` 과 `@NotNull` 어노테이션을 붙여서 사용하라.

</aside>

### 어노테이션을 활용하라.

- 아래와 같은 여러 어노테이션이 지원된다.
    - JetBrains(org.jetbrains.annotations의 @Nullable, @NotNull)
    - Android(androidx.annotation, com.android.annotations, android.support.annotations의 @nullable, @Nonnull
    - JSR-3O5(javax.annotation의 @Nullable, @CheckForNull, @Nonnull)
    - JavaX(javax.annotation의 @Nullable, @CheckForNull, @Nonnull)
    - FindBugs(edu.umd.cs.findbugs.annotations의 @Nullable, @CheckForNull,@PossiblyNull, @NonNull)
    - Reactive(io.reactivex.annotations의 @Nullable, @NonNull)
    - Eclipse(org.eclipse.jdt.annotation의@Nullable,@NonNull)
    - Lombok(lombok의 @NonNull)
- 대체적으로 JSR 305의  `@ParametersAreNonnullByDefault` 어노테이션 등을 활용해서 자바에서도 디폴트로 파라미터가 널이 아니라는 것을 보장할 수 있다.

## 플랫폼 타입은 안전하지 않다.

```java
public class JavaClass {
		public String getValue() {
				return null;
		}
}
```

```kotlin
fun startedType() {
		val value: String = JavaClass().value
		//...
		println(value.length)
}

fun platformType() {
		val value = JavaClass().value
		//...
		println(value.length)
}
```

- 위 `startedType` 함수와 `platformType` 함수는 둘 다 NPE가 발생한다.
- `startedType` 의 경우 자바에서 값을 가져오는 위치에서 NPE가 발생한다.
    - 잘못된 부분을 찾기 쉽다.
- `platformType` 의 경우 값을 활용할 때 NPE가 발생한다.
    - 플랫폼 타입으로 지정된 변수는 nullable일 수도 있고, 아닐 수도 있다.
    - 이러한 변수를 한두 번 안전하게 사용했더라도, 이후에 NPE를 발생시킬 가능성이 존재한다.
    - 타입 검사기가 검출해 줄 수 없다.

### 타입 추론을 사용하는 경우

```kotlin
interface UserRepo {
		fun getUserName() = JavaClass().value
}
```

- 이러한 경우 메서드의 inferred타입(추론된 타입)이 플랫폼 타입
- 누구나 nullable 여부를 지정할 수 있다.

```kotlin
class UserRepoImpl: UserRepo {
		override fun getUserName(): String? {
				return null
		}
}
fun main() {
		val repo: UserRepo = UserRepoImpl()
		val text: String = repo.getUserName() // 런타임에서 NPE 발생
		print("유저 이름의 길이는 ${text.length}")
}
```

- 사용하는 사람이 nullable이 아닐거라고 받아들이는 경우 NPE발생
- 위와같이 플랫폼 타입이 다른곳에서 사용되는 일은 굉장히 위험하다.

## 정리

- 다른 언어에서 온 nullable 여부를 알 수 없는 타입을 **플랫폼 타입**이라고 한다.
- 플랫폼 타입을 사용하는 코드는 해당 부분뿐이 아니라 이를 활용하는 곳까지 영향을 주는 위험한 코드다.
- 가능한 자바 생성자, 메서드, 필드에 nullable여부를 지정하는 어노테이션을 활용하는 것이 좋다.