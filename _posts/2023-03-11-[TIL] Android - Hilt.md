---
layout: "post"
title: "[TIL] Android - Hilt 정리"
subtitle: "Andoird hilt"
date:       2023-03-11
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
    - Hilt
---

# [Android] hilt 정리

## Hilt란?

- 의존성 주입을 보다 편 하게 구현할 수 있도록 도와주는 라이브러리다.
- Android에 컴포넌트에 맞는 컨테이너 및 생명주기를 자동으로 관리
- dagger를 기반으로 빌드됨

## 의존성 추가

```kotlin
buildscript {
    dependencies {
        classpath("androidx.navigation:navigation-safe-args-gradle-plugin:${libs.versions.androidxNavigation.get()}")
    }
}

plugins {
		...
    alias(libs.plugins.hilt) apply false
}
```

```kotlin
plugins {
  kotlin("kapt")
  id("com.google.dagger.hilt.android")
}

android {
  ...
}

dependencies {
  implementation("com.google.dagger:hilt-android:2.44")
  kapt("com.google.dagger:hilt-android-compiler:2.44")
}

// Allow references to generated code
kapt {
  correctErrorTypes = true
}
```

## Hilt 애플리케이션 클래스

- Application에 `@HiltAndroidApp` 어노테이션을 포함해야함
    
    ```kotlin
    @HiltAndroidApp
    class ExampleApplication : Application() { ... }
    ```
    
- 애플리케이션 수준의 컨테이너 및 Hilt의 코드 생성을 트리거 함
- 해당 애플리케이션 객체의 생명 주기에 연결되어 관련 의존 항목을 제공함

## Android 클래스에 의존성 주입

- `@AndroidEntryPoint` 어노테이션을 통해 Android 클래스에 의존성을 주입할 수 있습니다.(Hilt 구성요소를 생성)
- **Application은 `@HiltAndroidApp` 을 통해서, ViewModel은 `@HiltViewModel` 을 사용해야 함**
    - 지원 항목
        
        
        | Application |
        | --- |
        | ViewModel |
        | Activity with ComponentActivity Extands |
        | Fragment with androidx.fragment Extands |
        | View |
        | Service |
        | BroadcastReceiver |
- 구성요소에서 의존클래스를 주입받으려면 `@Inject` 를 통해 실행
    
    ```kotlin
    @AndroidEntryPoint
    class ExampleActivity : AppCompatActivity() {
    
      @Inject lateinit var analytics: AnalyticsAdapter
      ...
    }
    ```
    

## 의존성 주입을 위한 구성요소 제공

- 의존성을 주입하기 위해서는 Hilt가 해당 클래스가 필요로하는 종속 항목의 인스턴스를 제공하는 방법을 알아야 함
    - ex) `class Programmer(private val computer: Computer)` 를 주입 하기 위해서는 computer 에 대한 정보가 필요
- 해당 정보를 제공하는 방법은 `@inject` 어노테이션이 지정된 생성자의 매개변수가 해당 클래스의 종속 항목임을 Hilt에 알려줄 수 있음.
    - `class Programmer @inject constructor(private val computer: Computer)`
    - 위의 경우 computer의 인스턴스를 제공하는 방법도 알아야함.

## Hilt 모듈

- `@Module` 어노테이션이 지정된 클래스
- 왜 필요한지?
    - 제공 타입에 따라 생성자에서 생성자에서 의존성 주입을 할 수 없을 때가 있을 수도 있다.
        - interface - 실제 구현체를 알 수 없음
        - 외부 라이브러리의 클래스
    - 위와 같은 상황에 module을 이용해 힐트에게 제공하는 타입에 해당하는 인스턴스 정보를 알려줄 수 있다.
- `@installIn` 어노테이션을 지정해서 모듈을 사용하거나 설치할 Android 컴포넌트에게 알려야한다.

## @Binds 를 이용한 인터페이스 인스턴스 주입

- Interface의 실제 어떤 인스턴스를 제공하는지 Hilt에게 알려준다.
- Module내에서 `@Binds` 어노테이션을 붙인 **추상 함수**를통해 해당 정보를 제공한다.
    - 매개 변수를 통해 실제 주입할 구현체를 Hilt에게 알려준다.
    - 리턴 타입은 어떤 타입을 주입할지 Hilt에게 알려준다.
    
    ```kotlin
    interface Computer{
    	fun on()
    }
    
    class IntelComputer @Inject constructor(
    	...
    ) : Computer{
     ...
    }
    
    @Module
    @InstallIn(ActivityComponent::class)
    abstract class IntelModule {
    	@Binds
    	abstract fun bindIntelComputer(
    		IntelComputer : IntelComputer
    	) : Computer
    }
    ```
    

## @Provides를 이용한 인터페이스 인스턴스 주입

- interface 뿐만 아니라 외부 클래스에서 제공하는 클래스를 소유하지 않은 경우에도 유형을 제공할 수 없는 경우가 있다.
    - 빌더 패턴을 통해 인스턴스를 만드는 경우와 같이 예를들어 Retrofit, OkHttpClinet, Room 등이 있다.
- Module내에서 `@Provides` 어노테이션을 붙인 함수를 통해 해당 인스턴스에 대한 정보를 제공
    - 매개변수는 해당 유형의 의존 항목을 Hilt에게 알려준다.
    - 리턴 타입은 실제 주입할 구현체를 Hilt에게 알려준다.
    - 함수의 본문에서는 해당 유형의 인스턴스를 제공하는 방법을 Hilt에게 알려준다.
        - 해당 유형의 인스턴스를 제공해야 할 때마다 함수 본문을 실행함.
    
    ```kotlin
    @Module
    @InstallIn(ActivityComponent::class)
    object IntelModule {
    	
    	@Provides
    	fun provideIntelComputer(
    		...//intelComputer의 생성자에 필요한 매개변수
    	) : IntelComputer {
    		return IntelComputer(...)
    	}
    }
    ```
    

## @Qualifier를 통한 동일한 타입에 대한 여러 인스턴스 제공

- 주입하고자 하는 타입이 동일하지만 구현체를 달리하는 경우에 사용된다.
- Hilt에는 기본적으로 제공하는 한정자 어노테이션이 있다.
    - `@ApplicationContext`
    - `@ActivityContext`
- `@Qualifer` , `@Retention` 어노테이션을 통해 **한정자 어노테이션을 만들 수있다.**
- `@Binds` , `@Provides` 메서드와 한정자 어노테이션을 통해 **어떤 인스턴스를 제공할 지** Hilt에게 알려줄 수 있다.
    - 아래는 동일한 부모클래스를 갖지만 구현체가 다른 computer를 통해 각기 다른 인스턴스(Programmer)를 제공합니다.
    
    ```kotlin
    @Qualifer
    @Retention(AnnotationRetention.BINARY)
    annotation class IntelProgrammer
    
    @Qualifer
    @Retention(AnnotationRetention.BINARY)
    annotation class RyzenProgrammer
    
    @Module
    @InstallIn(SingleTonComponent::class)
    object ProgrammerModule {
    	
    	@IntelProgrammer
    	@Provides
    	fun providesIntelProgrammer(
    		intelComputer: IntelComputer,
    	) : Programmer {
    			return Programmer(
    				intelComputer,
    			)
    	}
    	@RyzenProgrammer
    	@Provides
    	fun providesRyzenProgrammer(
    		ryzenComputer: RyzenComputer,
    	) : Programmer {
    			return Programmer(
    				ryzenComputer,
    			)
    	}
    }
    
    @AndroidEntryPoint
    class IntelProgrammerActivity: AppCompatActivity() {
    	
    	@IntelProgrammer
    	@Inject
    	lateinit var programmer: Programmer
    }
    
    @AndroidEntryPoint
    class RyzenProgrammerActivity: AppCompatActivity() {
    	
    	@RyzenProgrammer
    	@Inject
    	lateinit var programmer: Programmer
    }
    ```
    

## Android Component를 위한 구성요소

| Hilt 구성 요소 | 대상 | 생성 위치 | 소멸 위치 |
| --- | --- | --- | --- |
| SingletonComponent | Application | Application.onCreate() | Application 소멸됨 |
| ActivityRetainedComponent | N/A(해당 사항 없음) | Activity.onCreate() | Activity.onDestroy() |
| ViewModelComponent | ViewModel | ViewModel 생성됨 | ViewModel 소멸됨 |
| ActivityComponent | Activity | Activity.onCreate() | Activity.onDestroy() |
| FragmentComponent | Fragment | Fragment.onAttach() | Fragment.onDestroy() |
| ViewComponent | View | view.super() | View 소멸됨 |
| ViewWithFragmentComponent | @WithFragmentBindings 가 지정된 View | view.super() | View 소멸됨 |
| ServiceComponent | Service | Service.onCreate() | Service.onDestroy() |


## 구성요소 계층 구조

![hiltsupport][Hilt_Support_Component]

## Hilt가 지원하지 않는 클래스에 의존성 주입

- 위와 같이 Android Component는 Hilt가 제공하지만 새로운 Component를 제공하고자 한다면 `@EntryPoint` 어노테이션을 통해 사용자 정의가 가능하다.
- `EntryPointAccessors` 의 적절한 정적 메서드를 사용하여 진입점에 액세스 할 수 있다.
- [[ 자세한 내용 ]](https://developer.android.com/training/dependency-injection/hilt-android?hl=ko#not-supported)

### 참고 자료

---

[Hilt를 사용한 종속 항목 삽입  |  Android 개발자  |  Android Developers](https://developer.android.com/training/dependency-injection/hilt-android?hl=ko)

[Dagger Hilt로 안드로이드 의존성 주입 시작하기](https://hyperconnect.github.io/2020/07/28/android-dagger-hilt.html)

[Hilt_Support_Component]:/assets/images/post/2023-03-11-hilt-supoort-component.png