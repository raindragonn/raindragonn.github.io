---
layout: "post"
title: "[TIL] Android - ADB를 통한 액티비티 스택 확인하기"
subtitle: "Check Activity Stack"
date: 2023-04-06
author: "raindragonn"
lang: ko
categories: [TIL - Android]
tags:
    - ADB
    - Activity
---

## ADB를 통한 액티비티 스택 확인하기

```bash
adb shell dumpsys activity activities | grep -i "Hist"

adb shell dumpsys activity | grep -i "PACKAGENAME"
```
