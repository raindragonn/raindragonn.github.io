---
layout: "post"
title: "[kotlin] kotlin null 안전 연산자"
subtitle: "null"
date:       2021-04-15
author: "raindragon"
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
    - null
---

# Null 안전 연산자

코틀린 에서는 Null에 대한 처리가 엄격합니다.

그에따라 제공하는 연산자 입니다.

|연산자|사용법|설명|
|---|---|---|
|?|val a: Int?| 변수 a의 데이터타입을 Nullable한 Int형으로 선언|
|?:|A ?: B| A가 null 이면 B 실헹|
|?.|b?.length|b가 null이면 null, null이 아니면length
|!!|A!!|A가 null이 아님을 선언 null 일 경우 NPE 에러 발생|
