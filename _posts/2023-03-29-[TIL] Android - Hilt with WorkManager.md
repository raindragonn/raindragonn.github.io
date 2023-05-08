---
layout: "post"
title: "[TIL] Android - Hilt with workmanager"
subtitle: "Hilt with WorkManager"
date: 2023-03-29
author: "raindragonn"
lang: ko
categories: [TIL - Android]
tags:
    - Hilt
    - WorkManager
    - Jetpack
---

## WorkManager를 Hilt와 함께 사용하는 방법

### 1. 의존성 추가

```kotlin
dependencies {
    implementation("androidx.hilt:hilt-work:1.0.0")
    // When using Kotlin.
    kapt("androidx.hilt:hilt-compiler:1.0.0")
    // When using Java.
    annotationProcessor("androidx.hilt:hilt-compiler:1.0.0")
}
```

### 2. @HiltWorker 및 @AssistedInject, @Assisted 이용

```kotlin
@HiltWorker
class ExampleWorker @AssistedInject constructor(
  @Assisted appContext: Context,
  @Assisted workerParams: WorkerParameters,
  workerDependency: WorkerDependency
) : Worker(appContext, workerParams) { ... }
```

### 3. Application에서 Configuration.Provider 인터페이스 구현 및 HiltWorkFactory를 주입받아 사용

```kotlin
@HiltAndroidApp
class ExampleApplication : Application(), Configuration.Provider {

  @Inject lateinit var workerFactory: HiltWorkerFactory

  override fun getWorkManagerConfiguration() =
      Configuration.Builder()
            .setWorkerFactory(workerFactory)
            .build()
}
```

위의 방법이 현재(2023-03-29) 공식 레퍼런스에서 나온 가이드이다.

## 2.6.0-alpha01 버전 이상 사용 시 추가 사항

- `androidx.startup` 을 사용해 WorkManager를 초기화를 함으로써 추가적으로 AndroidManifest에 프로바이더를 포함해야함.

```kotlin
<provider
    android:name="androidx.startup.InitializationProvider"
    android:authorities="${applicationId}.androidx-startup"
    tools:node="remove" />
```

### 참고 자료

---

[https://developer.android.com/training/dependency-injection/hilt-jetpack#workmanager](https://developer.android.com/training/dependency-injection/hilt-jetpack#workmanager)

[https://github.com/google/dagger/issues/2601#issuecomment-837816151](https://github.com/google/dagger/issues/2601#issuecomment-837816151)