---
layout: "post"
title: "[TIL] Android - Fragment에서 Back Pressed 처리하기"
subtitle: "프래그먼트에서 백버튼 동작"
date: 2023-04-13
author: "raindragonn"
lang: ko
categories: [TIL - Android]
tags:
    - Fragment
    - OnBackPressedDispatcher
---

## [Android] Fragment에서 Back Pressed 처리하기.

## 필요성

최근에는 앱을 만들다 보면 필연적으로 Fragment를 사용하게 되고 기획이나 디자인에 따라 뒤로가기 시 동작을 현재 Fragment 별로 다르게 해야하는 일이 많습니다. 예전에는 interface를 통한 리스너나 braodcastreceiver등을 이용해서 처리하기도 헀는데 실 개발시에는 가령 생명주기나 고려해야할 사항이 많아서 불편했습니다. 최근에는 `OnBackPressedDispatcher`를 통해 보다 편하게 구현할 수 있습니다.

## 활용법

확장함수를 통해 아래와같이 `OnBackPressedCallback`을 받아서 등록할 수 있다.

```kotlin
fun Fragment.addBackPressedCallBack(callBack:OnBackPressedCallback) {
    val activity = activity ?: return
    activity.onBackPressedDispatcher.addCallback(
        this,
        callBack
    )
    lifecycle.addObserver(object : LifecycleEventObserver {
        override fun onStateChanged(source: LifecycleOwner, event:Lifecycle.Event) {
            when (event) {
Lifecycle.Event.ON_RESUME -> {
                    callBack.isEnabled = true
                }
Lifecycle.Event.ON_PAUSE -> {
                    callBack.isEnabled = false
                }
Lifecycle.Event.ON_DESTROY -> {
                    callBack.remove()
                }
                else -> {}
            }
        }
    })
}

fun Fragment.defaultBackPressed() {
    activity?.onBackPressedDispatcher?.onBackPressed()
}

fun getOnBackPressedCallBack(
    enabled: Boolean = false,
    action: () -> Unit,
):OnBackPressedCallback=
    object : OnBackPressedCallback(enabled) {
        override fun handleOnBackPressed() {
            action.invoke()
        }
    }

fun getOnBackPressedCallBack(
    action: () -> Unit,
):OnBackPressedCallback=
    object : OnBackPressedCallback(false) {
        override fun handleOnBackPressed() {
            action.invoke()
        }
    }
```

사용하는 프래그먼트

```kotlin
class TestFragment : Fragment {
...
    private val _backPressedCallBack: OnBackPressedCallback by lazy {
        getOnBackPressedCallBack(::onBackPressed)
    }

    override fun onAttach(context: Context) {
        super.onAttach(context)
        addBackPressedCallBack(_backPressedCallBack)
    }
		private fun onBackPressed(){
			if(isWorking) {
				showDialog()
			} else {
				_backPressedCallBack.isEnabled = false
				defaultBackPressed()
			}
		}
...	
}
```

### 참고 자료

---

[https://readystory.tistory.com/186](https://readystory.tistory.com/186)

[https://developer.android.com/reference/androidx/activity/OnBackPressedDispatcher](https://developer.android.com/reference/androidx/activity/OnBackPressedDispatcher)