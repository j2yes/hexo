---
title: passport
date: 2019-02-17 17:11:42
tags:
- nodejs
- express
- passport
- authentication
categories:
- nodejs
---

## passport : http://www.passportjs.org/

> Simple, unobtrusive authentication for Node.js

패스포트는 인증을 위한 다양한 Strategy Package를 제공합니다. 아래 본문은 그 중 Local Strategy에 대해 설명합니다.

## local strategy

외부 시스템의 인증 연동없이 직접 구현하는 방식이며 사용자 상태 관리를 위해 세션을 사용합니다.
로그인, 리소스 호출에 대한 흐름을 정리하고 코드로 설명하겠습니다.

## 사전작업 


### 패스포트가 자동으로 생성해주는 함수 (변수)

- req.user : 로그인한 사용자 정보
- req.login(=logIn) : 로그인 함수 (세션에 사용자 정보를 저장)
- req.logout : 로그아웃 함수 (세션정보 비움)
- req.isAuthenticated : 요청 사용자의 인증여부 확인

### Session Storage 설정

- express 는 기본 session을 in-memory 방식으로 저장하기 때문에 memory leak 문제가 있음(production 환경에서 사용 X)
- memory leak 방지 등 몇 가지 목적으로 session-storage 사용 추천 (mysql, mongodb, redis 등 모두 사용 가능함)
- session 은 기본적으로 passport가 아닌 express에서 관리해주고 있음, passport에서 회원정보를 기본적으로 저장하는 req.user정보는 req.session.passport.user를 의미하고 response하는 시점에 세션정보를 갱신함(store를 따로 써도 자동으로 갱신해줌)
- response하는 시점에 session을 업데이트하기 때문에 문제가 발생하면 req.logIn을 이용해서 serialize를 명시적으로 해줄 수 있음

```javascript
const express = require('express') ;
const passport = require('passport'); 
const session = require('express-session'); 
const RedisStore = require('connect-redis')(session);
 
const app = express() ;
app.use(session({ 
  store: new RedisStore({
    url: config.redisStore.url
  }),
  secret: config.redisStore.secret,
  //resave false는 session이 갱신된 경우만 데이터를 업데이트하도록 함(만료시간값 갱신은 관련없음)
  resave: false,
  saveUninitialized: false
}));
//express 연결
app.use(passport.initialize()); 
//영속적인 로그인 세션 관리 (deserialize)
app.use(passport.session());
```

### passport 기본 함수 구현

#### passport id & pwd 체크 

```javascript
const passport = require('passport') 
const LocalStrategy = require('passport-local').Strategy
 
const user = { 
  username: 'test-user',
  password: 'test-password',
  id: 1
}
 
//post로 넘어오는 parameter명으로 변수명을 설정한다 (email, passwd) 
passport.use(new LocalStrategy({
    usernameField: 'email',
    passwordField: 'passwd'
  },
  function(username, password, done) {
    //findUser는 실제로 database에서 user를 조회해야 함
    findUser(username, function (err, user) {
      if (err) {
        return done(err)
      }
      //일차하는 user가 없는 경우 ( = 일차하는 id 없음 )
      if (!user) {
        return done(null, false, { message: 'Incorrect user information.' });
      }
      //조회한 user의 password가 입력된 password와 불일치 ( = 비밀번호 틀림 )
      if (!user.validPassword(password)) {
        return done(null, false, { message: 'Incorrect user information.' });
      }
      //예외 상황을 처리 완료 후 callback 두번째 parameter에 false가 아닌 값(searched user object)을 넘겨주면 로그인 성공
      return done(null, user)
    })
  }
))
 
 
//LocalStrategy의 parameter로 정의된 익명함수는 verify callback이다
//verify callback은 request가 포함한 credential을 분석하여 done함수를 발생시킨다.
//done 함수는 인증 실패/성공여부를 포함하여 실행된다.
//done 함수는 passport.authenticate (로그인 api에서 사용하는 함수) 의 두번째 parameter로 실행된다.
```

#### user serialize

```javascript
passport.serializeUser(function(user, done) {
    done(null, user);
});

// user를 두번째 인자로 넘기면 user 정보 전체를 session에 저장한다. user.id 로 id 정보만 저장하여 관리할 수 있음
```

#### user deserialize

```javascript
passport.deserializeUser(function(user, done) {
    done(null, user);
});

// session 에 저장된 user 정보를 deserialize 함, id만 관리한 경우 user 정보가 필요하면 DB에서 조회해야 함
```

#### authentication middleware

```javascript
function authenticationMiddleware () { 
  return function (req, res, next) {
    if (req.isAuthenticated()) {
      return next()
    }
    res.redirect('/')
  }
}
```

## login flow

flow

1. id 와 password를 post method로 login url로 요청
2. login router에서 passport.authenticate 함수를 이용해 strategy를 선택
3. 선택된 strategy에서 id, password 비교 (db 조회), 비교 결과를 callback 함수를 이용해서 passport.authenticate 두번째 인자 함수로 전달
4. serialize 를 통해 session에 user 정보 저장
5. callback 함수안에서 req.logIn 함수 실행
6. Set-Cookie 헤더에 세션키값을 저장하여  response
7. session storage를 사용하는 경우 response할 때 req.session 에 있는 user 정보를 storage에 저장

code

```javascript
app.get('/login', function(req, res, next) {
  passport.authenticate('local', function(err, user, info) {
    if (err) { return next(err); }
    if (!user) { return res.redirect('/login'); }
    //passport가 제공하는 login함수(=logIn)을 이용하여 login session 저장
    req.logIn(user, function(err) {
      if (err) { return next(err); }
      return res.redirect('/users/' + user.username);
    });
  })(req, res, next);
});

// logout 함수로 req.user 를 삭제하고 관련 login session을 비움
app.get('/logout', function(req, res){
  req.logout();
  res.redirect('/');
});
```

## resource authentication flow

flow 

1. endpont로 request 보냄
2. cookie이용해서 session 정보 조회
3. session 정보 deserialize 해서 user 정보 얻기
4. req.isAuthenticate 함수를 이용해서 인증된 요청인지 확인
5. 인증된 경우 / 인증되지 않은 경우에 따라 처리

code

```javascript
const passport = require('passport')
app.get('/resource', passport.authenticationMiddleware(), function(req, res){
    res.render('resource', { title: 'passport example' });
});
```

## 참조 링크

- https://blog.risingstack.com/node-hero-node-js-authentication-passport-js/
- https://github.com/passport/express-4.x-local-example/blob/master/server.js
- https://scotch.io/tutorials/easy-node-authentication-setup-and-local
- https://bcho.tistory.com/920