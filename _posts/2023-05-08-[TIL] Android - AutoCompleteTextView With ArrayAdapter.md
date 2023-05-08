---
layout: "post"
title: "[TIL] Android - AutoCompleteTextView With ArrayAdapter disable click(아이템 선택 안되도록 변경)"
subtitle: "AutoCompleteTextView With ArrayAdapter"
date: 2023-05-08
author: "raindragonn"
lang: ko
categories: [TIL - Android]
tags:
    - AutoCompleteTextView
    - ArrayAdapter
---

## 해당하는 아이템이 없는 경우 특정 텍스트 보여주기.

`AutoCompleteTextView` + `ArrayAdapter`를 통해 검색어를 포함하는 항목을 보여주고 검색어가 없는경우 “해당하는 항목이 없습니다”이 뜨도록 해두었다.

```kotlin

...
override fun getCount(): Int {
  val count = super.getCount()
  return if (count == 0) {
      1
  } else {
      count
  }
}

override fun getItem(position: Int): String? {
    return kotlin.runCatching {
        super.getItem(position)
    }.getOrElse { NOT_FOUND_TEXT }
}

override fun getView(position: Int, convertView: View?, parent: ViewGroup): View {
	...
	val text = getItem(position) ?: NOT_FOUND_TEXT
	view.text = text
	return view
} 

companion object {
	private const val NOT_FOUND_TEXT = "해당하는 항목이 없습니다."
}
...

```

## 해당하는 항목이 없는 경우 클릭 이벤트 막기

“해당하는 항목이 없습니다” 를 클릭 시에도 `AutoCompleteTextView` 가 채워지는 문제가 있었다. (getView에서 enabled를 변경해도 클릭됨)

아이템별 클릭 이벤트를 조정하기 위해서는 `areAllItemsEnabled` 와 `isEnabled` 를 오버라이드해서 원하는대로 동작하도록 할 수 있다.

```kotlin
...

override fun areAllItemsEnabled(): Boolean {
	return false
}

override fun isEnabled(position: Int): Boolean {
    return getItem(position) != NOT_FOUND_TEXT
}
...
```

`areAllItemsEnabled` 를 통해 기본적으로 모두 선택이 안되도록 변경,

`isEnabled` 를 통해 아이템의 값이 `NOT_FOUND_TEXT` 인 경우에는 동작하지 않도록 변경

### 참고 자료

---

[https://stackoverflow.com/questions/22142578/how-to-disable-an-autocomplete-item](https://stackoverflow.com/questions/22142578/how-to-disable-an-autocomplete-item)