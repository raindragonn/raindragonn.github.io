---
layout: "post"
title: "[TIL] Android - Mac에서 APK 디컴파일 하기"
subtitle: "APK Decompile"
date: 2023-05-09
author: "raindragonn"
lang: ko
categories: [TIL - Android]
tags:
    - APK
    - Decompile
---

# [Android] Mac에서 APK 디컴파일 하기

암호화가 잘 적용이 됐는지 확인하기 위해 APK를 Decompile 해봐야할 때가 있다.

## 준비물

1. jd-gui
2. apktool, dex2jar
3. 확인하고자 하는 APK

### 1. jd-gui 준비하기

1. Clone받아서 build → build/distributions/에 확인
    
    ```bash
    git clone https://github.com/java-decompiler/jd-gui.git
    ...
    cd jd-gui/
    ./gradlew build
    ```
    
2. [링크](https://github.com/java-decompiler/jd-gui/releases/tag/v1.6.6)에서 다운받기

### 2. apktool, dex2jar 준비하기

```bash
brew install apktool
brew install dex2jar
```

### 3. APK to Jar

```bash
apktool d -s -o decompile app-debug.apk
```

- 위 명령어를 이용하면 decompile이라는 폴더가 생기고 그 안에 classes.dex와 같은 파일이 보인다.

```bash
cd decompile

d2j-dex2jar classes.dex
```

- 위 명령어를 이용하면 dex파일을 .jar 파일이 생긴다.

### 4. JD-GUI를 통해 확인.

- jd-gui를 통해 3에서 생성한 jar파일을 확인.
- JD-GUI.app을 받았지만 실행이 안되는경우
    
    ```bash
    //jd-gui.app이 있는 곳으로 이동
    
    java -jar JD-GUI.app/Contents/Resources/Java/jd-gui-1.6.6-min.jar
    ```
    

### 참고 자료

---

[https://medium.com/@yonghan_89267/apk-decompile-fcf64c6e74a5](https://medium.com/@yonghan_89267/apk-decompile-fcf64c6e74a5)

[https://github.com/java-decompiler/jd-gui/issues/391](https://github.com/java-decompiler/jd-gui/issues/391)