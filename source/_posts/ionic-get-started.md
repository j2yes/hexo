---
title: ionic-get-started
date: 2019-02-04 15:49:46
tags:
- mobile
- ionic
- angular
categories:
- ionic
---
### 프로젝트 구조

![폴더 구조](/images/ionic/folder_structure.png "folder_structure")

- src/app : app.module.ts가 entry point ( index.html에서 ion-app에 표시될 root component가 entry point 인 app.module파일에 지정되어 있다. )
- src/pages : 화면 페이지 단위로 구분
- src/asset : 이미지 등 

### ionic view life cycle

http://blog.ionic.io/navigating-lifecycle-events/

### 페이지 파일 구성

한페이지에 ts, html, scss 파일 구조를 유지하는게 좋음

예) hello-ionic/
- hello-ionic.html
- hello-ionic.scss
- hello-ionic.ts
  ```javascript
  import { Component } from '@angular/core';
  
  @Component({
    //selector 는 필수는 아니지만 특정 페이지에서 기본 스타일 적용에 유용함
    selector: 'page-hello-ionic',
    templateUrl: 'hello-ionic.html'
  })
  export class HelloIonicPage {
    constructor() {
  
    }
  }
  ```

### ionic navigation

ionic은 navigationController를 이용해서 flexible하게 페이지를 이동할 수 있다.

```sbtshell
#ion-nav 태그에 root로 지정된 페이지가 가장 상위 페이지
<ion-nav [root]="rootPage"></ion-nav>
```

페이지 이동 (push, pop)

```javascript
class OtherPage {
  constructor(public navCtrl: NavController) {}

  //다른 페이지로 이동하기
  goToOtherPage() {
    //push another page onto the history stack
    //causing the nav controller to animate the new page in
    this.navCtrl.push(OtherPage);
  }

  //뒤로가기 (ion-navbar 태그를 페이지에 포함하고 있는 경우, 백버튼에 대한 이벤트 처리가 자동으로 처리됨)
  goBack() {
    this.navCtrl.pop();
  }
}
```

#### 탭 페이지이동 처리

탭으로 구성된 화면은 각각의 탭이 최상위 화면이 되어 각 탭마다 고유한 아이디를 갖고 있는 navigationController를 이용함

### component 정리하기 (http://ionicframework.com/docs//api/)

#### http 통신하기

http 를 이용하는 방법은 두가지
- @angular/http 사용
- @ionic-native/http 사용 
- 차이점 : https://forum.ionicframework.com/t/what-is-the-difference-between-angular-and-native-http/73095

@ionic-native/http 사용 예시

- 설치
  ```sbtshell
  #설치
  ionic plugin add --save cordova-plugin-http
  npm install --save @ionic-native/http
  ```
- 설정
  ```javascript
  // provider에 추가하기
  // app.module.ts 파일에 provider로 추가하고 사용해야 함
  // ...
  
  providers: [
      StatusBar,
      SplashScreen,
      HTTP,
      {provide: ErrorHandler, useClass: IonicErrorHandler}
    ]
  
  // ...
  ```

@angular/http 사용 예시

- 설정
  ```javascript
  // app.module.ts 파일에 모듈 추가하고 사용해야 함
  // ...
  import { HttpModule } from '@angular/http';
  
  // ...
  imports: [
    BrowserModule,
    HttpModule,
    IonicModule.forRoot(MyApp)
  ],
  ```

#### 참조문서

- https://ionicframework.com/docs/native/http/
- http://blog.ionic.io/10-minutes-with-ionic-2-calling-an-api/


#### whitelist 설정 (http://www.gajotres.net/ionic-2-making-rest-http-requests-like-a-pro/)

- 설치
  ```sbtshell
  cordova plugin add cordova-plugin-whitelist
  ```
- proxy 설정하기
  ```sbtshell
  #프로젝트의 ionic.config.json 파일에 설정
  {
    "name": "auction",  "app_id": "",  "type": "ionic-angular",  "proxies": [
      {
        "path": "/apt",      "proxyUrl": "http://openapi.molit.go.kr/OpenAPI_ToolInstallPackage/service/rest/RTMSOBJSvc/getRTMSDataSvcAptTradeDev"    }
    ]
  }
  ```

### typescript 기본

https://www.typescriptlang.org/docs/handbook/basic-types.html

### angular2 기본 문법

반복문

```angular2html
<li *ngFor="let hero of heroes” (click)="onSelect(hero)>
    <span class="badge">{{hero.id}}</span> {{hero.name}}
</li>
```

조건문

```angular2html
<div *ngIf="selectedHero">
  <h2>{{selectedHero.name}} details!</h2>
  <div><label>id: </label>{{selectedHero.id}}</div>
  <div>
    <label>name: </label>
    <input [(ngModel)]="selectedHero.name" placeholder="name"/>
  </div>
</div>
```

동적 스타일

```angular2html
<li *ngFor="let hero of heroes"
  [class.selected]="hero === selectedHero"
  (click)="onSelect(hero)">
  <span class="badge">{{hero.id}}</span> {{hero.name}}
</li>
```

@Injectable() 사용하기 : https://angular.io/docs/ts/latest/guide/dependency-injection.html

