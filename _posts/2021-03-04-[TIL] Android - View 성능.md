---
layout: "post"
title: "[TIL] Android - View 성능 올리기"
subtitle: "Android 성능 개선"
author: "raindragon"
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
---


# View 성능 개선 방안

### 1. 오버드로우 줄이기

**오버드로우를 줄이면 불필요한 GPU리소스를 줄이고 퍼포먼스를 끌어올릴 수 있다.**

#### `오버드로우(OverDraw)`란?

- 같은 픽셀에 여려번 덧 칠하는 것을 의미한다.

#### <span style="color:black">1.1 불필요한 배경 중첩이 일어나는 경우</span>

```xml
    <LinearLayout
        android:id="@+id/ll_parent"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@color/black"
        android:orientation="vertical">

        <LinearLayout
            android:id="@+id/ll_child"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:background="@color/black"
            android:orientation="vertical" >
            ...
        </LinearLayout>
        
    </LinearLayout>
```

위의 XML과 같이 부모 뷰와 자식 뷰의 배경색이 중첩되는 경우 불필요한 오버드로우가 생길 수 있다.

#### <span style="color:black">1.2 투명도 사용 삼가하기</span>

1. 배경색을 회색으로 설정한 경우

2. 배경색을 검은색으로, 투명도를 50으로 준 경우

위의 두 예시는 보기에 같은 색상일 수 있습니다.

하지만 2번의 경우 부가적인 연산이 들어가므로 불필요한 투명도 사용을 삼가는 것이 좋습니다.


#### <span style="color:black">1.3 View의 간소화 및 갱신 빈도 낮추기</span>

뷰는 다음과 같은 생명 주기를 가집니다

![ViewLifeCycle][ViewLifeCycle]

커스텀 뷰를 만든다고 할 때 주의할 점으로

1. `onDraw()`는 수시로 호출될 수 있기에 메모리를 할당하거나 객체를 초기화하는 작업을 삼갑니다.
2. 무분별한 `invalidate()`, `requestLayout()` 호출 삼가기


#### <span style="color:black">1.4 레이아웃 계층 구조를 가능한 `간단하게` 하기</span>

레이아웃 계층이 깊을수록 연산량이 증가합니다.

1. ConstraintLayout 최대한 사용하기

`ConstraintLayout`은 뷰간의 관계와 제약사항을 통해 레이아웃 계층의 깊이를 깊게 만들지 않는 것이 목적입니다.

### 2. RecyclerView 성능 향상

#### <span style="color:black">2.1 아이템 갱신 최소화하기</span> 

`Adapter.notifyDataSetChanged()` : 전체 아이템에 대해 갱신이 일어납니다. (변경되지 않은 아이템도 갱신)

위의 메서드를 이용하기보다는 부분적으로 아이템을 갱신할 수 있는 아래 메서드를 이용 하는 게 좋습니다

- notifyItemChanged
- notifyItemInserted
- notifyItemRemoved
- notifyItemMoved
- notifyItemRangeChanged
- notifyItemRangeInserted
- notifyItemRangeRemoved

#### <span style="color:black">2.2 DiffUtil,ListAdapter 사용하기</span> 

**DiffUtill이란?**

두 리스트 차이를 계산하고 변경된 부분만 갱신하도록 도와주는 유틸클래스

`DiffUtill.ItemCallback<T>` 사용시 구현해야하는 메서드

- `areItemsTheSame()` : 두 아이템이 같은 아이템인지 검사합니다.

- `areContentsTheSame()` : 두 아이템의 내용이 같은지 검사합니다.




---

참고 자료

[네이버 테크 콘서트](https://tv.naver.com/v/15353556)

[ViewLifeCycle]:/assets/images/post/2021-03-04-ViewLifeCycle.png "View LifeCycle"