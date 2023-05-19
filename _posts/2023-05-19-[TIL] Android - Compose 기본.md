---
layout: "post"
title: "[TIL] Android - Compose 기본"
subtitle: "Compose Basic codelab"
date: 2023-05-19
author: "raindragonn"
lang: ko
categories: [TIL - Android]
tags:
    - Android
    - Codelab
    - Compose
---

## [Android] Compose Codelab

## 컴포즈란?

- 안드로이드에서 UI 개발을 간소화하기 위해 설계된 최신 툴킷
- UI 개발을 단순화 하며 반응형 프로그래밍모델을 사용한다.
- **xml을 사용하지 않고 kotlin을 사용한다.**
- 선언형 UI 프레임워크로 데이터를 UI 레이어로 변환해주는 일련의 함수를 호출하여 UI를 구성한다.
- 데이터가 변경된 경우 해당 함수를 자동으로 호출하여 실시간으로 뷰를 업데이트한다.

## @Composable

- 일반 함수와 똑같이 생겼지만 `@Composable` 어노테이션을 통해 해당 함수가 컴포저블 함수로 사용한다.
- 새 UI를 구성요소를 생성할 때 해당 함수를 사용한다.
- 컴포저블 함수는 다른 다른 컴포저블 함수를 호출할 수 있다.
    - 컴포저블 함수가 아닌곳에서`(@Composable` 이 없는 일반 함수 등)에서는
    - `Functions which invoke @Composable functions must be marked with the @Composable annotation` 라는 에러메시지를 확인할 수 있다.

## 컴포즈 사용하기.

### Activity에서 사용하기.

- `androidx.activity.compose.setContent` 함수를 통해 해당 액티비티를 컴포즈로 전환한다.

```kotlin
public fun ComponentActivity.setContent(
    parent: CompositionContext? = null,
    content: @Composable () -> Unit
) {
    val existingComposeView = window.decorView
        .findViewById<ViewGroup>(android.R.id.content)
        .getChildAt(0) as? ComposeView

    if (existingComposeView != null) with(existingComposeView) {
        setParentCompositionContext(parent)
        setContent(content)
    } else ComposeView(this).apply {
        // Set content and parent **before** setContentView
        // to have ComposeView create the composition on attach
        setParentCompositionContext(parent)
        setContent(content)
        // Set the view tree owners before setting the content view so that the inflation process
        // and attach listeners will see them already present
        setOwners()
        setContentView(this, DefaultActivityContentLayoutParams)
    }
}
```

- 위 함수의 content 파라미터를 통해 컴포저블 함수를 호출할 수 있다.

## Surface

- 컴포저블 함수의 일종으로 컨테이너 컴포저블이다.
- 배경생상, 콘텐츠색상 등의 변경 가능
- 여러가지 값을 통해(파라미터) UI를 구성할 수 있다(surface의 값을 수정할 수 있다.)

```kotlin
@Composable
fun Surface(
    modifier: Modifier = Modifier,
    shape: Shape = RectangleShape,
    color: Color = MaterialTheme.colors.surface,
    contentColor: Color = contentColorFor(color),
    border: BorderStroke? = null,
    elevation: Dp = 0.dp,
    content: @Composable () -> Unit
) {
    val absoluteElevation = LocalAbsoluteElevation.current + elevation
    CompositionLocalProvider(
        LocalContentColor provides contentColor,
        LocalAbsoluteElevation provides absoluteElevation
    ) {
        Box(
            modifier = modifier
                .surface(
                    shape = shape,
                    backgroundColor = surfaceColorAtElevation(
                        color = color,
                        elevationOverlay = LocalElevationOverlay.current,
                        absoluteElevation = absoluteElevation
                    ),
                    border = border,
                    elevation = elevation
                )
                .semantics(mergeDescendants = false) {}
                .pointerInput(Unit) {},
            propagateMinConstraints = true
        ) {
            content()
        }
    }
}
```

## Theme

- 스타일을 적용할 수 있다.
- 레이어의 가장 위에 있는 컴포저블만 래핑해도 하위 컴포저블도 영향을 받는다.

## Preview(미리보기)

- `@preview`, `@Composable` 어노테이션을 통해 Android Studio에 내장된 미리보기를 사용할 수 있다.
- `@Preview` 어노테이션을 여러개 사용해서 다양한 미리보기를 만들 수도 있다.
- 해당 함수는 매개변수를 받지 않아야한다.
- 에디터 영역에서 split이나 Design 이용
    
    ![Untitled](/assets/images/post/2023-05-19-preview.png)
    

## Modifier [[ link ]](https://developer.android.com/jetpack/compose/modifiers?hl=ko)

- 컴포저블을 생성하고 중첩할 수 있을 뿐아니라 컴포저블에 `Modidifier`를 추가하여 컴포저블을 수정할 수있다.
- 요소를 꾸미거나 동작을 추가 및 수정할 수 있다.
- 다양한 종류가 있다.
    - padding, focsable, clickable, background sizer 등

## Column

- 하위 컴포저블을 수직으로 배치할 수 있다.
- 기본적으로 Column은 하위요소가 필요한 만큼만 공간을 사용한다.(wrap_content)

## Row

- 하위 컴포저블을 수평으로 배치할 수 있다

## LazyColumn, LazyRow

- 현재 화면안에 있는 컴포저블만(화면에 표시된것만) 렌더링한다.
- `item`, `items` 를 통해 구성이 가능하다

## Remember, State

- 일반적인 값(일반 변수)의 변경으로는 컴포저블이 업데이트 되지 않는다.
- `remember` 와 , `mutableStateOf()` 를 통해 컴포저블엑게 재구성하라고 알려줄 수 있습니다.
- remember는 내부에서 생성된 값이 컴포저블이 재구성될 때마다 재설정되지 않도록 한다.
    - 특정 데이터를 사용함을 의미.
- remember.value 를 통해 새로운 변수를 사용하는경우도 사용이 가능하다.
    - 다만, 복잡한 로직이 들어가는 경우 CPU 사이클을 더 많이 사용하기에 remember 블록 내부에 작성하는 편이 더 좋다.
- by 키워드를 통해(델리게이트) remember를 사용할 경우 `.value`를 사용하지 않고 값을 바로 사용한다.

## Hoisting

- 어느 항목에서 사용하는 상태를 다른곳에서도 알 필요가 있을 때가 있다.(공통적 상위 항목에 있어야 함)
- 상태를 상위 항목으로 이동시키는 과정을 `State Hosting`상태 호이스팅이라고 한다.
- 이는 상태가 중복되는 버그를 막고 컴포저블을 재사용하는데 도움이 된다.
- 상위 항목에서 사용하지 않는 상태는 호이스팅해서는 안된다.

## RememberSavable

- 원래의 상태를 기억하는 방법.
- 일반 remember의 경우 테마를 바꾸는것 처럼 구성이 변경되어 Activity가 다시 그려지는 경우에 이전 데이터를 잃어버리지마 RememberSavable은 이전 데이터를 유지한다.

### 참고 자료

---

[https://www.youtube.com/watch?v=k3jvNqj4m08&t=24s](https://www.youtube.com/watch?v=k3jvNqj4m08&t=24s)

[https://developer.android.com/codelabs/jetpack-compose-basics?hl=ko#0](https://developer.android.com/codelabs/jetpack-compose-basics?hl=ko#0)