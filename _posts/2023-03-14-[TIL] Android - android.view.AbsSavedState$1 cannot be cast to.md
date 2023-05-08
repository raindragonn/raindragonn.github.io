---
layout: "post"
title: "[TIL] Android - android.view.AbsSavedState$1 cannot be cast to 에러"
subtitle: "AttributeSet"
date:       2023-03-14
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
    - View
    - Error
---

# [Android] java.lang.ClassCastException: android.view.AbsSavedState$1 cannot be cast to android.widget.ProgressBar$SavedState


## 문제 발생

커스텀뷰를 만들던 중 `ClassCastException` 가 발생하는 경우가 있었다. 

구글링 중 스택오버 플로우나 많은 블로그애서 동일한 ID를 사용하는 경우 해당 문제가 발생한다는 글이 많이 보였다.

코드를 아무리 봐도 동일한 아이디를 사용하는 뷰는 있지 않았다.

## 해결 방법

해결 방법을 찾아보던 중 부모 뷰의 `AttributeSet` 객체를 자식에게 동일하게 사용하지 말라는 글을 보았다. [[ Link ]](https://stackoverflow.com/a/50865837)

코드 상에서 **커스텀뷰 안에서 사용하는 자식 뷰에게 부모의 attrs 를 넘기지 않으니 에러가 사라졌다.**

에러가 발생하는 시점은 커스텀 뷰의 `onSaveInstanceState()` 메서드가 실행되면서 발생했다.

해당 메서드는 어떤일을 할까?

> Hook allowing a view to generate a representation of its internal state that can later be used to create a new instance with that same state. This state should only contain information that is not persistent or can not be reconstructed later. For example, you will never store your current position on screen because that will be computed again when a new instance of the view is placed in its view hierarchy. Some examples of things you may store here: the current cursor position in a text view (but usually not the text itself since that is stored in a content provider or other persistent storage), the currently selected item in a list view.
> 

해당 메서드에 대한 설명이다.

대략 뷰가 동일한 상태의 새 인스턴스를 만드는 경우 사용할 데이터를 생성할 수 있도록 한다는듯 하다.

ex) 목록에서 현재 선택한 항목, ProgressBar의 progress 등

`AttributeSet`을 그대로 전달한 경우 **자식 뷰와 부모의 뷰가 동일한 ID를 가지므로 문제**가 된다는 내용이였다.

나는 커스텀 뷰 내부에서 세가지의 뷰를 사용했는데 동일한 atts를 사용해서 해당 뷰들의 아이디가 동일하게 설정되어 문제가 된 듯 했다. 

아래는 attrs를 같이 사용했을때와 부모만 사용한 경우 각 뷰의 ID를 로그로 확인한 경우다.

![부모의 attrs를 같이 사용한 경우][before]

부모의 attrs를 같이 사용한 경우

![부모만 attrs를 사용한 경우][after]

부모만 attrs를 사용한 경우

또 다른 방안으로는 `ViewCompat.generateViewid()`를 통해 각 뷰마다 아이디를 지정해도 되는듯 하다.

## AttributeSet은 어떤일을 할까?

> A collection of attributes, as found associated with a tag in an XML document.

해당 클래스에 대한 간략한 설명이다.

XML레이아웃 파일에서 뷰에 지정된 속성들의 집합.

AttributeSet은 생성자에 전달되며 뷰가 `inflation`될 때 사용, 해당 객체를 통해 속성값을 가져올 수 있다.

[before]:/assets/images/post/2023-03-14-before.png
[after]:/assets/images/post/2023-03-14-after.png