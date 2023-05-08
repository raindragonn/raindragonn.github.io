---
layout: "post"
title: "[Android] Tip - Mac에서 핸드폰 미러링 하기"
subtitle: "Scrcpy"
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
    - TIP
    - Scrcpy
---


나는 Mac으로 Android 앱개발을 하면서 에뮬레이터 보다는 실기기를 주로 사용을 하고있다.  
코딩을 하다가 핸드폰을 들고 기능 확인을 한다던가 하면 이리저리 움직이는게 너무 불편해서 미러링 프로그램을 찾아보다 설치와 사용도쉬운 [scrcpy][scrcpy]를 찾았다.

Mac OS 에서 설치법
===

패키지 관리자인 [homebrew][homebrew]가 필요하다.

없을경우 아래의 명령어를 iterm이나 터미널에서 입력해서 설치하자.

    
    $ /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

`1. Scrcpy의 설치는 아래의 명령어 입력후`
   
   
    $ brew install scrcpy

`2. homebrew 버전에 맞게 아래와 같이 입력하면 된다.`


    # Homebrew >= 2.6.0
    $ brew install --cask android-platform-tools

    # Homebrew < 2.6.0
    $ brew cask install android-platform-tools


사용법
===


맥에 디바이스 연결후 iterm 이나 터미널에서 입력하면된다 :


    $ scrcpy

    (명령어 확인)
    $ scrcpy --help


## 단축키

 | 실행내용                                |   단축키                       |   단축키 (macOS)
 | -------------------------------------- |:----------------------------- |:-----------------------------
 | 전체화면 모드로 전환                      | `Ctrl`+`f`                    | `Cmd`+`f`
 | window를 1:1비율로 전환하기(픽셀 맞춤)   | `Ctrl`+`g`                    | `Cmd`+`g`
 | 검은 공백 제거 위한 window 크기 조정  | `Ctrl`+`x` \| _Double-click¹_ | `Cmd`+`x`  \| _Double-click¹_
 |`HOME` 클릭                        | `Ctrl`+`h` \| _Middle-click_  | `Ctrl`+`h` \| _Middle-click_
 | `BACK` 클릭                      | `Ctrl`+`b` \| _Right-click²_  | `Cmd`+`b`  \| _Right-click²_
 | `APP_SWITCH` 클릭                 | `Ctrl`+`s`                    | `Cmd`+`s`
 | `MENU` 클릭                       | `Ctrl`+`m`                    | `Ctrl`+`m`
 | `VOLUME_UP` 클릭                   | `Ctrl`+`↑` _(up)_             | `Cmd`+`↑` _(up)_
 | `VOLUME_DOWN` 클릭                | `Ctrl`+`↓` _(down)_           | `Cmd`+`↓` _(down)_
 | `POWER` 클릭                      | `Ctrl`+`p`                    | `Cmd`+`p`
 | 전원 켜기                               | _Right-click²_                | _Right-click²_
 | 미러링 중 디바이스 화면 끄기    | `Ctrl`+`o`                    | `Cmd`+`o`
 | 알림 패널 늘리기               | `Ctrl`+`n`                    | `Cmd`+`n`
 | 알림 패널 닫기            | `Ctrl`+`Shift`+`n`            | `Cmd`+`Shift`+`n`
 | 디바이스의 clipboard 컴퓨터로 복사하기      | `Ctrl`+`c`                    | `Cmd`+`c`
 | 컴퓨터의 clipboard 디바이스에 붙여넣기     | `Ctrl`+`v`                    | `Cmd`+`v`
 | Copy computer clipboard to device      | `Ctrl`+`Shift`+`v`            | `Cmd`+`Shift`+`v`
 | Enable/disable FPS counter (on stdout) | `Ctrl`+`i`                    | `Cmd`+`i`





[scrcpy]: https://github.com/Genymobile/scrcpy
[homebrew]: https://brew.sh/index_ko
