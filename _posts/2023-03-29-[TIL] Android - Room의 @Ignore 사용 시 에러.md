---
layout: "post"
title: "[TIL] Android - Room 의 @Ignore 사용 시 에러"
subtitle: "Room"
date: 2023-03-29
author: "raindragonn"
lang: ko
categories: [TIL - Android]
tags:
    - Room
    - Jetpack
---


## 문제 발생

Room에서 Entity의 프로퍼티 중 칼럼으로 사용하고 싶지 않은 경우 `@Igore` 를 사용하면 된다고 [공식 레퍼런스](https://developer.android.com/training/data-storage/room/defining-data?hl=ko#ignore)에 나와있다.

하지만 단순히 `@Ignore` 어노테이션만 사용 시 

> error: Entities and POJOs must have a usable public constructor. You can have an empty constructor or a constructor whose parameters match the fields (by name and type).
> 

위와 같은 에러를 만나게 된다.

## 해결 방법

`@Ignore` 어노테이션을 적용한 필드가 없는 생성자를 만들면 된다.

```kotlin
@Entity("user")
data class User(
	@PrimaryKey val id: Int,
	val nickName: String,
	@Ignore val number: String?,
) {
	constructor(id: Int, nickName: String) : this(id, nickName, null)
}
```

**Room은 Entity와 일치하는 생성자만을 가질 수 있기 때문이다.**

- Ignore를 통해 특정 프로퍼티를 무시한 경우 Room에서는 해당 프로퍼티를 포함하지 않는 생성자를 필요로 한다.

### 참고 자료

---

[https://github.com/android/architecture-components-samples/issues/421#issuecomment-533217060](https://github.com/android/architecture-components-samples/issues/421#issuecomment-533217060)