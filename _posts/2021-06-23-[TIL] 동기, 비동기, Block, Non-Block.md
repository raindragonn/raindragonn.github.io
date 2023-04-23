---
layout: "post"
title: "[TIL] 동기와 비동기, Blocking, Non-Blocking"
subtitle: "Sync, Async, Blocking, Non-Blocking"
date:       2021-06-26
author: "raindragon"
banner:
  image: "/assets/images/base/banner.jpg"
  opacity: 0.618
  height: "38vh"
  min_height: "38vh"
  heading_style: "font-size: 2.25em; font-weight: bold; text-decoration: underline"
  subheading_style: "color: gold"
lang: ko
categories: [TIL - CS]
tags:
    - CS
---

동기와 비동기, Blocking과 Non-Blocking을 많이 보고 듣게 되지만

해당 개념들을 생각해보자면 두루뭉술하게 안다고 해야 할까? 헷갈리기도 하고

명확하게 설명하기는 힘든 느낌이 들어 정리해보려 합니다.


## Blocking과 Non-Blocking

**자신의 제어권이 관점입니다.**

`Blocking` : 단어의 뜻 그대로 막는 것을 생각하면 됩니다. 자신의 작업을 진행 중에 다른 주체의 작업을 시작하면 다른 작업이 끝날 때까지 `기다렸다`가 자신의 작업을 시작하는 것으로 `제어권이 넘어간 경우`를 의미합니다.

`Non-Blocking` : 다른 주체의 작업에 `관련 없이` 자신의 작업을 하는 것으로 제어권이 `자신에게 있는 경우`를 의미합니다.


## 동기와 비동기 (Synchronous / Asynchronous)

**두 개 이상의 작업이 동시에 이뤄질 때 작업의 종료 및 수행결과에 대한 처리가 관점입니다.**

`Synchronous` : 작업을 동시에 수행하거나, 동시에 끝나거나, `끝나는 동시에 시작함`을 의미합니다. (다른 작업의 수행 결과 및 종료를 신경 씀)

`Asynchronous` : 시작, 종료가 일치하지 않으며, 끝나는 동시에 시작하지 않음을 의미합니다.(`다른 작업의 수행 결과 및 종료를 신경 쓰지 않음 ex) Call back`)


## 동기/비동기 와 Blocking/Non-Blocking의 조합의 예시

> Ex) 협업하는 디자이너에게 이미지 파일을 요구한 경우

**1. Blocking / Sync**

1. 개발자 : 이미지 파일좀 주실수 있을까요?

2. 디자이너 : 여기 잠깐만 계세요. USB에 담아 드릴게요.
  
3. 개발자 : ?!(기다림)

   - `제어권이 넘어가며(Blocking)`

4. 디자이너 : 여기용 (이미지 파일 작업 후 USB 전달)

5. 개발자 : 작업을 이어감

   - `결과를 받아서 처리함(Sync)`

**2. Non-Blocking / Sync**

1. 개발자 : 이미지 파일좀 주실수 있을까요?

2. 디자이너 : 작업하고 슬랙으로 보내드릴게요. 다른일 하고 계세요~

3. 개발자 : 네 (다른일 중)

   - `제어권이 넘어가지 않으며(Non-Blocking)`

4. 디자이너 : (이미지 작업중..)

5. 개발자 : 끝났나요?

6. 디자이너 : 아니요.

7. 개발자 : 끝났나요?

   - `지속적으로 결과를 확인하여(Sync를 위해)`

8. 디자이너 : 네 보냈습니다.

9. 개발자 : 이미지 파일을 받아서 작업함.

   - `결과를 받아서 처리함(Sync)`

**3. Blocking / Async**

> 일반적이지 않은 케이스에 속합니다. 보통 Non-Block/Async를 하려다가 잘못된 경우가 많다고 합니다.

1. 개발자 : 나중에 사용될 이미지 파일이 있을까요?

2. 디자이너 : 여기 잠깐만 계세요. 지금 USB에 담아 드릴게요.

3. 개발자 : ?!(지금은 필요없는데..)

   - `제어권이 넘어가며(Blocking)`

4. 디자이너 : 여기용 (이미지 파일 작업 후 USB 전달)

5. 개발자 : 아 네..

   - `결과에 상관하지 않음(Async)`

**4. Non-Blocking / Async**

1. 개발자 : 이미지 파일좀 주실수 있을까요?

2. 디자이너 : 작업하고 슬랙으로 보내드릴게요. 다른일 하고 계세요~

3. 개발자 : 네

4. 디자이너 : (이미지 작업중)

5. 개발자 : (다른일 중)

   - `제어권이 넘어가지 않으며(Non-Blocking)`

6. 디자이너 : 보냈습니다~

7. 개발자 : 네~ (이미지는 나중에 해야지)

   - `결과에 상관하지 않음(Async)`




___

참고자료

- [https://musma.github.io/2019/04/17/blocking-and-synchronous.html](https://musma.github.io/2019/04/17/blocking-and-synchronous.html)

- [우아한Tech-멍토님테코톡](https://www.youtube.com/watch?v=IdpkfygWIMk)
