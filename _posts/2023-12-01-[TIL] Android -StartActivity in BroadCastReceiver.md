---
layout: "post"
title: "[Android] StartActivity in BroadCastReceiver"
subtitle: "SYSTEM_ALERT_WINDOW"
date: 2023-12-01
author: "raindragonn"
lang: ko
categories: [TIL - Android]
tags:
  - Android
  - StartActivity
  - BroadCastReceiver
  - AlarmManager
---

## [Android] StartActivity in BroadCastReceiver

## 필요상황

알람앱을 제작중 `AlarmManager`를 통해 지정한 시간에 `BroadCastReceiver`를 실행하고 해당 `Receiver`에서 `AlarmActivity`를 실행하도록 했다.

Android 12부터 [제한사항](https://developer.android.com/about/versions/12/behavior-changes-12?hl=ko#notification-trampolines)으로 인해 백그라운드(BroadCastReceiver)에서 `Context.startActivity()`를 실행 하더라도 액티비티는 띄워지지 않았다..

`FullScreenIntent`([Link](https://developer.android.com/reference/android/app/Notification.Builder#setFullScreenIntent(android.app.PendingIntent,%20boolean))) 를 사용하면 일부 원하는대로 동작이 가능하지만 현재 화면이 켜져있는 경우에는 헤드업 알림으로 나오게 된다.

현재 제작중인 앱은 추후 유튜브 영상을 알람으로도 사용이 가능하도록 지원할 예정이라 Notification(알림)을 통한 알람은 가능한 사용하고 싶지 않았다.

## 해결법

현재 서비스중인 여러 알람앱들을 확인한 결과 [SYSTEM_ALERT_WINDOW](https://developer.android.com/reference/android/Manifest.permission#SYSTEM_ALERT_WINDOW) 권한이 없을 경우 원활한 동작을 지원하지 않는 다는것을 확인했다.

`SYSTEM_ALERT_WINDOW` 권한을 받은 이후 Receiver에서 StartActivity가 정상적으로 동작하는것을 확인했다.

### 참고 자료

---

https://developer.android.com/about/versions/12/behavior-changes-12?hl=ko#notification-trampolines