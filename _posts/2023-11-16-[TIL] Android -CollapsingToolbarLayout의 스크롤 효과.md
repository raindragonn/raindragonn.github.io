---
layout: "post"
title: "[Android] CoordinatorLayout + CollapsingToolbarLayout 사용 시 스크롤에 따른 색상 변경 없애기"
subtitle: "CollapsingToolbarLayout"
date: 2023-11-16
author: "raindragonn"
lang: ko
categories: [Android]
tags:
  - Android
  - CoordinatorLayout
  - CollapsingToolbarLayout
---

## [Android] CoordinatorLayout + CollapsingToolbarLayout 사용 시 스크롤에 따른 색상 변경 없애기

```xml
<CoordinatorLayout>
	<AppbarLayout>
		<CollapsingToolbarLayout
		...
		app:contentScrim="color/transparent"
		>
		...
		...
		</CollapsingToolbarLayout>
	</AppbarLayout>	
</CoordinatorLayout>
```

CollapsingToolbarLayout의 `contentScrim` 옵션을 사용하면 된다.

### CollapsingToolbarLayout의 Content Scrim이란?

- 스크롤이 특정 임계값을 도달하면 해당 속성으로 정의된 리소스를 통해 표시하거나 숨기는게 가능하다.

### Scrim이란?

- 콘텐츠를 덜 눈에 띄게 만들기 위해 뷰 표면에 적용할 수 있는 임시 처리를 의미한다.
- [[ 링크 ]](https://m2.material.io/design/environment/surfaces.html#attributes)

### 참고 자료

---

https://stackoverflow.com/questions/37911696/what-is-scrim-in-collapsibletoolbarlayout