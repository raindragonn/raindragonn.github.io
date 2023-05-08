---
layout: "post"
title: "[TIL]Android - BuildConfig 상수 추가"
subtitle: "BuildConfig"
date:       2021-05-31
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
    - BuildConfig
---

## BuildConfig

앱을 빌드하면 `BuildConfig` 클래스가 생성 됩니다.

이 클래스를 통해 `Application id, Build Type, Version code, Version Name`을 확인할 수 있습니다.

해당 클래스에 정의된 상수들을 불러와 디버그시에만 로그를 찍는다든지

api 키값을 변경 할 수 있습니다.

이러한 값들은 App수준의 `build.gradle`의 defaultConfig 에서 가져오게 됩니다.

ex)build.gradle

```gradle
defaultConfig {
        applicationId "com.raindragonn.buildconfig"
        minSdkVersion 23
        targetSdkVersion 30
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }
```

### BuildConfig에 상수 추가 및 사용하기

#### build.gradle 에 직접 추가

원하는 상수를 추가하려면 

`defaultConfig 블록 안`에 `buildConfigField` 를 통해 선언할 수 있습니다.

```gradle
android {
    //...
    defaultConfig {
        // ...
        buildConfigField "String", "API_KEY", '"jkdsfl12ndfs1"'
    }

}
```

빌드 타입에 따라 (debug 와 release) 구분하고 싶은 경우는

`buildTypes 에서 각각 선언하면 됩니다.`

```gradle
android {
    //...
    buildTypes {
        release {
            //...
            buildConfigField "String", "API_KEY", '"릴리즈용 키"'    
        }
        debug {
            //...
            buildConfigField "String", "API_KEY", '"디버그용 키"'    
        }
        
    }

}
```

String 외의 타입
```gradle
buildConfigField("boolean", "USE_MOCK", "true")
buildConfigField("int", "MOCKING_DATA_SIZE", "10")
```


#### properties 파일에 선언 후 가져오기


**1.local.properties 에 선언한 값 가져오기**

`ex) local.properties`
```gradle
SOME_API_KEY = "lkasdfjafdkl2"
```

ex) build.gradle(app)
```gradle
plugins { 
    //...
} 
Properties properties = new Properties()
properties.load(project.rootProject.file('local.properties').newDataInputStream())
android {
     //...
     defaultConfig {
         //...
         buildConfigField "String", "SOME_API_KEY", properties['SOME_API_KEY'] 
         }
    }
```


**2.gradle.properties 에 선언한 값 가져오기**

git에 코드를 공유할때 api키 와 같은 민감한 정보들을 넣어둔 local.properties를 ignore하고 올리기도 합니다.

`ex) gradle.properties`
```gradle
SOME_API_KEY = "lkasdfjafdkl2"
```

ex) build.gradle(app)
```gradle
plugins { 
    //...
} 
android {
     //...
     defaultConfig {
         //...
         buildConfigField "String", "SOME_API_KEY", project.properties["SOME_API_KEY"]
    }
```

#### 사용하기

```kotlin
fun initViews() {
    if(BuildConfig.DEBUG) {
        binding.debugView.isVisible = true
        Log.d(TAG,"디버그 디버그~")
    }
}
```
---

참고

[[안드로이드] local.properties에 API Key값 숨기기](https://devvkkid.tistory.com/201)

[Android - BuildConfig 정보 읽기 및 상수 추가](https://codechacha.com/ko/android-buildconfig/)


[[Android] BuildConfig 변수 생성/사용하기](https://hello-bryan.tistory.com/143)
