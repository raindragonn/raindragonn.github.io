---
layout: "post"
title: "[TIL] Android - TextView에 클릭이벤트 추가하기 (Spanable)"
subtitle: "Spanable"
date: 2023-04-17
author: "raindragonn"
lang: ko
categories: [TIL - Android]
tags:
    - TextView
    - Spanable
---

## 필요성

- 로그인 화면과 같이 약관을 확인할 수 있는 화면에서 텍스트에 일정부분에만 밑줄과 함께 링크를 제공하거나 특정 액션을 취해야할 때가 있다.

## 방식

- 텍스트뷰의 확장함수를 통해 `SpannableStringBuilder`, `ClickableSpan`을 통해 액션을 추가한다.
- 아래 참고자료 링크를 확인해 보다 다양한 방식을 만들어 사용이 가능하다.

## 사용법

### 확장함수

```kotlin
fun TextView.applyClickableSpan(
    string: String,
    maps: Map<String, () -> Unit>,
    underlineText: Boolean = true,
    @ColorRes highlightsColor: Int = R.color.transparent,
) {
    runCatching {
        movementMethod = LinkMovementMethod.getInstance()
        highlightColor = context.getColor(highlightsColor)

        text = SpannableStringBuilder(string)
            .apply {
                for ((clickableString, clickEvent) in maps) {
                    val startIndex = this.indexOf(clickableString)
                    val endIndex = startIndex + clickableString.length
                    setSpan(
                        object : ClickableSpan() {
                            override fun updateDrawState(ds: TextPaint) {
                                ds.isUnderlineText = underlineText
                            }

                            override fun onClick(widget: View) {
                                clickEvent.invoke()
                            }
                        },
                        startIndex,
                        endIndex,
                        Spannable.SPAN_EXCLUSIVE_EXCLUSIVE
                    )
                }
            }
    }.onFailure { e ->
        Timber.w(e)
        text = string
    }
}
```

### 실 사용

```kotlin
val serviceTermsString = getString(R.string.sns_login_terms)
val linkText = getString(R.string.sns_login_terms_link)

tvServiceTerms.applyClickableSpan(
	serviceTermsString,
  mapOf(
	  linkText to {
      Toast.makeText(this@LoginActivity, linkText, Toast.LENGTH_SHORT).show()
    }
  )
)
```

### 참고 자료

---

[https://developer.android.com/develop/ui/views/text-and-emoji/spans](https://developer.android.com/develop/ui/views/text-and-emoji/spans)

[https://developer.android.com/reference/android/text/Spanned](https://developer.android.com/reference/android/text/Spanned)