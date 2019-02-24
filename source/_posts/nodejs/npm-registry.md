---
title: npm-registry
date: 2019-02-24 21:04:36
tags:
- nodejs
- npm
- nexus
categories:
- nodejs
---

## private node package manger 

www.npmjs.com 에서도 개인이 한 달에 7$를 지불하면 private 패키지를 관리할 수 있다.
여건만 된다면 사설 저장소를 설치해서 관리하는 것도 나쁘지 않다.
미러링이나 캐쉬로 사용가능하기 때문에 private 으로 관리할 수 있다는 점 외에 장점이 많다.
사설 패키지 매니저로 `sinopia`와 `nexus`를 설치해보고 노드 프로젝트에 설정해보자.

## sinopia
https://www.npmjs.com/package/sinopia

npm 엔드포인트 수정
```sbtshell
$ npm config get registry
https://registry.npmjs.org/
$ npm set registry http://localhost:4873
$ npm config get registry
http://localhost:4873/
```

## nexus
https://www.sonatype.com/

### 설치 & 실행

1. nexus를 설치하는 방법은 2가지가 있다.
  - docker 사용
  - 그냥 설치
2. 그냥 설치해 보자 … 우선 다운로드 받는다. https://www.sonatype.com/download-oss-sonatype
3. java 8이 설치되어 있어야 한다.
4. 아카이브 해제하면 폴더 2개가 나온다
5. sonatype-work 는 저장소다,
6. 다른 폴더의 안의 bin 폴더에서 nexus 파일을 실행
7. http://localhost:8081/ 로 접속
8. default admin account [admin / admin123]

### repository 생성

![](/images/npm-registry/create_repo.png "create_repository")

- 저장소 목록에서 url 컬럼의 버튼을 누르면 저장소 주소가 copy됨
- repository 타입
  - group : repository를 하나로 묶을 수 있는 repo
  - hosted : 저장소 직접 호스팅
  - proxy : 외부 저장소 prox
-  npm 저장소 만들기
  - proxy에 공식 저장소 주소 넣기 : `https://registry.npmjs.org/`
  - hosted 만들기
  - group 으로 hosted proxy 묶기

### nexus npm저장소 계정만들기

![](/images/npm-registry/create_acc.png "create_account")

### 저장소 realms 설정

![](/images/npm-registry/set_realms.png "set_realms")

### user role 설정 
처음 anonymous 권한 설정이 읽기에 열려 있기 때문에 필요에 따라 anonymous user의 role 변경이 필요함

![](/images/npm-registry/set_role.png "set_role")

### nexus npm 저장소 사용하기

- module publish 관련 설정
  - .npmrc파일 생성
    ```sbtshell
    #_auth=<username>:<password>
    #base64 인코딩
    _auth=YWRtaW46YWRtaW4xMjM=
    ```
  - package.json 수정
    ```sbtshell
    {
      ...
    
      "publishConfig": {
        "registry": "http://your.nexus.repository.com/repository/npm-private/"
      }
    }
    ```
- install repository 지정
  - npm config command 이용
  - .npmrc 파일에 설정정보 추가
    ```sbtshell
    registry=http://your.nexus.repository.com/repository/npm-group/
    ```

## reference
- http://www.sonatype.org/nexus/2017/02/14/using-nexus-3-as-your-repository-part-2-npm-packages/
- https://help.sonatype.com/display/NXRM3
- https://gs.saro.me/#!m=elec&jn=774 
- https://kingbbode.github.io/posts/nexus-3xx-maven-npm
- https://blog.sonatype.com/using-nexus-3-as-your-repository-part-2-npm-packages