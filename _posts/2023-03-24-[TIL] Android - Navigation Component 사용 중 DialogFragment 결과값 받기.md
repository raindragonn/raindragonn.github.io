---
layout: "post"
title: "[TIL] Android - Navigation Component 사용 중 DialogFragment 결과값 받기"
subtitle: "NavigationComponent"
date: 2023-03-24
author: "raindragonn"
lang: ko
categories: [TIL - Android]
tags:
    - Navigation
    - Jetpack
    - fragment
---

## [Android] Navigation Component 사용 중 DialogFragment 결과값 받기

Navigation Component의 backStack을 활용해서 데이터를 주고 받는게 가능하다.

예시로 ExamFragment에서 ExamDetailDialogFragment로 `navigate` 한 경우

백스택에 대한 정보를 `ArrayDeque<NavBackStackEntry>` 타입으로 ExamDetailDialogFragment, ExamFragment의 값을 가진다.(`NavController.backQueue` 를 통해 확인 가능)

이를 이용해 savedState에 접근해 값을 관찰하거나 값을 지정할 수 있다.

## 확장 함수를 이용하는 방법 - 코드

```kotlin
fun <T> Fragment.getNavigationResult(key: String) =
    findNavController()
        .currentBackStackEntry
        ?.savedStateHandle
				?.apply { remove<T>(key) }
        ?.getLiveData<T>(key)

fun <T> Fragment.setNavigationResult(key: String, data: T) {
    findNavController()
        .previousBackStackEntry
        ?.savedStateHandle
        ?.set(key, data)
}
```

### 사용예시

```kotlin
class ExamFragment: Fragment() {
	...
	private fun showDialog() {
		getNavigationResult<String>(ExamDetailDialogFragment.EXAM_KEY)
			?.observe(viewLifecycleOwner) {test ->
				...
			}
		val action = ...
		findNavController().navigate(action)
	}
}

class ExamDetailDialogFragment: DialogFragment() {
	...
	override fun onDismiss(dialog: DialogInterface) {
		setNavigationResult(EXAM_KEY, "Test")
	}
	...
	
	companion object { 
		const val EXAM_KEY: String = "exam_key"
	}
}

```

### 주의사항

- [popUpTo 및 popupToInclusive](https://developer.android.com/guide/navigation/navigation-navigate?hl=ko#pop) 를 이용한 경우 백스택이남지않아  crash가 날 수 있다.
- 확장함수에서 currentBackStackEntry, previousBackStackEntry로 savedStateHandle을 가져오다보니 해당 확장함수의 **호출 시점을 잘 판단하고 사용해야한다.**
    - 현재 예시처럼 `getNavigationResult` 를 `navigate()` 호출하기 전에 사용하면 잘 동작하지만 순서를 바꿔 `navigate()` 후  `getNavigationResult` 를 사용하는 경우 동작하지 않는다.

### 참고 자료

---