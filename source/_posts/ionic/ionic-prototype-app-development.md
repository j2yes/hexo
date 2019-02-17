---
title: ionic-prototype-app-development
date: 2019-02-04 16:14:38
tags:
- mobile
- ionic
categories:
- ionic
---
> p2p 금융상품 모아서 보여주기

이번에 p2p 금융상품을 모아서 보여주는 앱을 만들어 보기로 했습니다.
p2p에 투자하려다 보면 사이트마다 들어가서 투자상품을 확인하는게 너무 번거로워서 시작했는데, 이미 있더군요...

### 시작하기
ionic이 문제인건지 angular가 문제인지 업데이트만 되면 앱을 수정해야 합니다.

업데이트하고 버전에 맞게 수정하는데 너무 많은 시간을 써서 최신 stable로 시작하고 버전을 fix하고 사용했습니다.

node와 ionic 설치를 끝내고, 프로젝트를 생성합니다.

````sbtshell
ionic start MyIonicProject super
````

super로 만들면 왠만한 기능의 템플릿이 포함되어 있어서 `복붙`이 너무 편해요.

나중에 필요없는 파일 지우는것도 일이긴 하지만 빨리, 쉽게 만들기 위해서 super로 생성했습니다.

폴더 구조도 잘 잡혀있어서 처음 ionic 프로젝트를 접해보는 분께 추천드립니다. 

프로젝트 생성이 완료되었으면 간단히 실행해봅시다.

````sbtshell
ionic serve
````

브라우저에서 정상적으로 실행된다면 기본적인 설정은 끝입니다.

#### 기능 정의하기
어떤 화면이 필요할까...

이번 앱의 목적은 여러 p2p 사이트의 딜 목록을 모아서 보여주는 것입니다.

대표적인 p2p 사이트를 들락날락.. 공통적으로 보이는 factor가 수익률(이자율)과 금액 소진율정도입니다.

이미지가 있으면 좋겠지만 이미지는 없는 경우도 많아서 공통으로 넣기 애매해 보입니다.

사실 상환방법에 따라 실수익률이 많이 달라지기 때문에 상환방식도 목록에서 보여주고 싶지만, 해당 정보가 p2p사이트에서 상세로 빠져있는 경우가 많아 빼기로 했습니다.
 
그래서 목록에서 보여지는 정보를 다섯가지 항목으로 구성하기로 했습니다.

***목록에서 보여지는 정보***
- 사이트로고
- 금액
- 수익률
- 기간
- 소진율(optional)

목록 화면은 사이트 검색이 가능하게하고, 화면 상단에 대쉬보드 기능을 겸할 수 있는 요약 정보를 보여주기로 합니다.

한페이지로 끝내버리기 위한.. 화면 구성입니다.

이제 생각한대로 만들기 시작해봅시다.

#### 플러그인 설치

대쉬보드는 검색 사이트의 누적 데이터를 보여주도록 합시다.
https://www.joshmorony.com/adding-responsive-charts-graphs-to-ionic-2-applications/
- 누적 투자금 (파이차트)
- 누적 수익률
- 기준금리(http://www.bok.or.kr/baserate/baserateList.action?menuNaviId=1927)

중간에 소진율을 보여주기 위한 progress bar 도 추가하고..
```sbtshell
#https://www.joshmorony.com/build-a-simple-progress-bar-component-in-ionic-2/
ionic g component ProgressBar
```

상세화면을 띄우기 위해서 in-app-browser도 설치
```sbtshell
#https://ionicframework.com/docs/native/in-app-browser/
ionic cordova plugin add cordova-plugin-inappbrowser
npm install --save @ionic-native/in-app-browser
```

대충 화면 작업은 끝났습니다. 처음 예상한 화면이랑 조금 다르긴 한데, 유사한 결과물을 만들었습니다.
(통계화면과 목록화면을 나누고 사이트별 검색은 셀렉트박스를 이용했습니다.)

##### 딜 목록화면

![딜 목록화면](/images/ionic/deal_list.webp "딜 목록화면")

##### 통계화면

![통계화면](/images/ionic/p2p_statistics.webp "통계화면")

#### api 만들기
node를 이용해서 api 서버를 만들고, 스크랩핑할 수 있도록 작업합니다.

필수정보만 디비스키마를 만들어서 필요한 정보를 저장
```sql
CREATE TABLE `p2p`.`deal` (
  `id` INT NOT NULL AUTO_INCREMENT COMMENT '딜 아이디 ',
  `siteCode` VARCHAR(45) CHARACTER SET 'utf8' NOT NULL,
  `dealCode` VARCHAR(45) CHARACTER SET 'utf8' NOT NULL,
  `dealTitle` VARCHAR(100) CHARACTER SET 'utf8' NOT NULL,
  `dealUrl` VARCHAR(500) CHARACTER SET 'utf8' NOT NULL,
  `dealAmount` INT NOT NULL COMMENT '딜 금액 ',
  `dealRevenueRate` FLOAT NOT NULL COMMENT '수익률 ',
  `dealMonthPeriod` INT NOT NULL COMMENT '딜 기간 (월 기준) ',
  `dealRunoutRate` FLOAT NOT NULL COMMENT '딜 소진율 ',
  `updatedAt` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '갱신일시\n',
  `createdAt` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '생성일시\n',
  PRIMARY KEY (`id`));

CREATE TABLE `p2p`.`site` (
  `siteCode` VARCHAR(45) CHARACTER SET 'utf8' NOT NULL COMMENT '사이트 코드 ',
  `siteName` VARCHAR(100) CHARACTER SET 'utf8' NOT NULL COMMENT '사이트 이름 ',
  `siteUrl` VARCHAR(500) CHARACTER SET 'utf8' NOT NULL COMMENT '사이트 주소 ',
  `siteLogo` VARCHAR(500) CHARACTER SET 'utf8' NOT NULL COMMENT '사이트 로고 ',
  PRIMARY KEY (`siteCode`));
```
