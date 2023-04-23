---
layout: "post"
title: "[TIL] Android - ListAdapter.submitList로 뷰가 업데이트 되지 않는 이유(스크롤 해야 뷰가 업데이트 되는 이유)"
subtitle: "Android SetHasStableIds"
date:       2023-03-20
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
    - RecyclerView 
---

# [Android] listAdapter submitList로 뷰가 업데이트 되지 않는 이유(스크롤을 해야 뷰가 업데이트 되는 이유)

## 문제 발생

RecyclerView + ListAdapter로 리스트를 보여주고 있었다.

어떤 트리거에 따라 주기적으로 리스트를 업데이트 해야했는데 

분명 데이터는 변경되었고 `ListAdapter.submitList` 로  넘겨서 업데이트 되었는데 실제로 뷰를 확인해보면 스크롤을 이동했다가 온 경우에만 리스트가 변경 되었다.

구글링을 통해 여러가지 이유를 확인했지만 데이터의 주소 값이 같은 이슈는 아니였다.

## 해결 방법

내 상황에서 문제는 성능을 높이기 위해 adapter의 `setHasStableIds(true)` 를 이용했는데  adapter에서 `getItemId` 를 override 하는 과정에서 `position.toLong()` 으로 사용한것이 문제였다.

`setHasStableIds` 를 true로 하는 경우 Adapter에서 position을 통해 아이템의 아이디에 따라 같은 아이디의 경우`onBindViewHolder`의 동작을 줄일 수 있다.

하지만 해당 방법은 `getItemId` 의 결과값을 아이템의 고유값으로 설정하지 않고 position과 같은 값을 이용하는 경우 문제가 될 수 있다.

새로운 리스트로 업데이트 한다고 해도 **같은 position에서는 동일한 아이디값을 가져** `onBindViewHolder`가 제대로 불려지지 않아 즉각적으로 리스트가 업데이트 되지 않는 문제가 생길 수 있다.
