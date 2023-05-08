---
layout: "post"
title: "[TIL] Android - keystore"
subtitle: "APK 키스토어 확인"
date: 2023-05-08
author: "raindragonn"
lang: ko
categories: [TIL - Android]
tags:
    - Apk
    - KeyStore
---

# [Android] APK 키스토어 확인

### APK의 서명된 fingerprint 확인

```bash
keytool -printcert -jarfile **app-debug.apk**
```

keytool -printcert - jarfile [확인하고자 하는 APK]

### 키스토어의 fingerprint 확인

```kotlin
keytool -list -keystore app.keystore
```

keytool -list -keystore [확인하고자 하는 키스토어]

### 참고 자료

---

[https://stackoverflow.com/questions/11331469/how-do-i-find-out-which-keystore-was-used-to-sign-an-app](https://stackoverflow.com/questions/11331469/how-do-i-find-out-which-keystore-was-used-to-sign-an-app)

