---
title: flutter
date: 2019-01-18 14:49:46
tags:
- mobile
- flutter
categories:
- flutter
---

## flutter 설치하기 

flutter는 windows, mac, linux에 설치할 수 있다.

### 시스템 요구사항
- windows 7 이상
- 400mb 이상 disk
- powershell 5.0 이상 / git 사용가능 상태

### SDK 설치(https://flutter.io/setup-windows/)
1. SDK 다운로드 받기
2. 다운받은 zip파일의 압축을 풀고 적당한 path에 옮기기
3.  `flutter` 디렉토리에 있는 `flutter_console.bat`파일을 더블클릭해서 실행

### 설정에 필요한 flutter 기본 command
```jshelllanguage
#setup 완료를 위해 필요한 디펜던시 확인하기
flutter doctor
#flutter 설정 확인하기
flutter config
```

### 기타
- flutter 업그레이드 하기 (https://flutter.io/upgrading/)
- flutter path 설정하기 (일반적인 윈도우 프롬프트에서 flutter 커맨드를 사용하려면)
  - 윈도우 환경설정 path에 `flutter\bin;` 위치 추가하기
  
## android studio 설치

IDE는 vs code, intellij, android studio를 지원한다. (https://flutter.io/get-started/editor/)

android studio, flutter plugin 설치하기
- file -> settings -> plugins, search flutter and install (dart plugin 같이 설치필요)


## 프로젝트 생성하기 (android studio)

1. flutter 프로젝트 생성
![new flutter project](/images/flutter/new_flutter_prj.png "create flutter project")
2. flutter application 선택 
3. flutter sdk path 는 압축해제한 flutter 디렉토리 지정
4. finish 하면 아래처럼 생성된 프로젝트를 확인할 수 있습니다.
![main.dart](/images/flutter/created_flutter_app.png "created flutter")

## device 설정하기 (android studio)

- flutter는 Android API 16이상 device가 필요합니다.
- tools -> AVD Manager -> Create Virtual Device...
- tools -> SDK Manager (생성한 device api에 맞는 sdk를 설치해야 합니다.)
- 가상 기기를 실행하면 android studio 에서 프로젝트를 실행할 수 있습니다.
![virtual device](/images/flutter/virtual_device.png "virtual device")
- command로 디바이스 확인하고 실행하기
  - 디바이스 확인 : `flutter devices`
  - 디바이스 실행 : `flutter run`

## flutter package repository
https://pub.dartlang.org/flutter/

## flutter 공부하기 
- free course : https://www.udacity.com/course/build-native-mobile-apps-with-flutter--ud905