---
layout: "post"
title: "[TIL] Android - ndk 에러 java.lang.UnsatisfiedLinkError"
subtitle: "java.lang.UnsatisfiedLinkError"
date:       2021-05-07
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
    - ndk
    - Error
---

최근에 개인앱을 업데이트 하기 위해 프로덕션에 새 버전을 만들어 올렸는데 검토중에

크래시리틱스 알람이 오는걸보니 구글에서 테스트 중에 발생한 것 같습니다.


> `Fatal Exception: java.lang.UnsatisfiedLinkError`<br>`dalvik.system.PathClassLoader`

### 발생 원인

 api key들을 보다 디컴파일에서 보다 안전하게 하기 위해 Ndk를 이용했습니다. [(방식)][ndk]

 해당 에러는 ndk 사용시 사용 기기(cpu)에 따라 지원하는 [ABI][abi]가 나눠지는데

 기본적으로는 각 ABI용 .so파일이 lib폴더속에 각각 존재 하지만
 
 `해당 기기가 .so 파일이 없는곳을 참조하려 할때 발생` 한다고 합니다. 

 오류가 발생한 apk나 aab 파일을 Analyzer로 확인해 보니 `mips` 폴더에만 so 파일이 없었습니다.

 ![analyzer_before][analyzer_before]


### 해결법

지원하는 ABI를 미리 정하고 모두 통일시켜서 같은 ABI만 사용할 수 있습니다.[link][특정abi]

애플리케이션에서 지원하는 ABI 세트를 제한하려면 `abiFilters`를 사용합니다.
  
예를 들어 64비트 ABI만 빌드하려면 `build.gradle`에서 다음 구성을 설정합니다.

 - ~~공식 레퍼런스를 보니 64비트 기기를 중점으로 맞추면 될거같습니다.~~
 > 64비트 기기는 32비트 변형도 지원합니다. arm64-v8a 기기를 예로 들면 이 기기는 armeabi 및 armeabi-v7a 코드도 실행할 수 있습니다. 다만 애플리케이션이 arm64-v8a를 대상으로 한다면 애플리케이션의 armeabi-v7a 버전을 실행하는 기기를 사용하는 것보다 64비트 기기에서 더욱 잘 실행된다는 점에 유의해야 합니다.

```gradle
android {
    defaultConfig {
        ndk {
            abiFilters 'arm64-v8a', 'x86_64'
        }
    }
}
```

![analyzer_after][analyzer_after]


2021-05-08 추가사항

 * 위의 명령어를 통해 abi를 설정한다면 해당하지 않는 cpu를 가진 핸드폰에서는 지원하지 않기 때문에 잘 선택해서 사용해야 합니다.
 
 * [보다 나은 설명][chacha]

---

참고

[https://creaby.tistory.com/11](https://creaby.tistory.com/11)

[공식 레퍼런스](https://developer.android.com/ndk/guides/abis?hl=ko)




[ndk]: https://medium.com/@abhi007tyagi/storing-api-keys-using-android-ndk-6abb0adcadad

[abi]: https://developer.android.com/ndk/guides/abis?hl=ko#android-platform-abi-support

[특정abi]: https://developer.android.com/ndk/guides/abis?hl=ko#gc

[analyzer_before]:/assets/images/post/2021-05-07-before.png 

[analyzer_after]:/assets/images/post/2021-05-07-after.png 

[chacha]: https://codechacha.com/ko/support-64-bit-app-in-android/