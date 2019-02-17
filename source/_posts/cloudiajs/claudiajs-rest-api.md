---
title: claudiajs-rest-api
date: 2019-01-19 22:22:17
tags:
- aws
- lambda
- claudiajs
categories:
- claudiajs
---
### claudiajs : https://claudiajs.com

1. make project
2. install claudia api module
3. modify package.json script command
4. make api.js file
5. make api gateway
6. example code

### make lambda project

```jshelllanguage
mkdir example
cd example
npm init
```

### install claudia api module : https://claudiajs.com/claudia-api-builder.html

```jshelllanguage
npm install claudia-api-builder -S
```

### modify package.json script command

package.json
```json
...
  "scripts": {
    "start": "claudia create --name rest-api-test --region ap-northeast-2 --api-module api --profile claudia",
    "deploy": "claudia update --profile claudia"
  },
...
```

### make a api.js file : 
- https://github.com/claudiajs/claudia-api-builder/blob/master/docs/api.md#api-definition-syntax
- https://github.com/claudiajs/claudia-api-builder/blob/4f5c30df0365812765806ae2f9fd97e7a1287ed9/docs/api.md

```javascript
var ApiBuilder = require('claudia-api-builder'),
	api = new ApiBuilder(),
	superb = require('superb');

module.exports = api;

api.get('/greet', function (request) {
	return request.queryString.name + ' is ' + superb();
});
```

### make api gateway : https://claudiajs.com/tutorials/hello-world-api-gateway.html

execute command

```jshelllanguage
#claudia create --name [lambda function name, api gateway name] --region [region name] --api-module [main javascript file name] --profile [profile name]
npm run start
```

you can see below console as result

saving configuration
```json
{
  "lambda": {
    "role": "rest-api-test-executor",
    "name": "rest-api-test",
    "region": "ap-northeast-2"
  },
  "api": {
    "id": "XXXXXXXX",
    "module": "api",
    "url": "https://XXXXXXXX.execute-api.ap-northeast-2.amazonaws.com/latest"
  }
}
```

you can connect to `api.url`

### example code

[github](https://github.com/j2yes/lambda-rest-claudia)

### 인증관련 Reference
- 카카오 : https://github.com/bskim/gamingonaws2017_serverless
- setting authorization to call api by aws iam : https://docs.aws.amazon.com/ko_kr/apigateway/latest/developerguide/api-gateway-control-access-using-iam-policies-to-invoke-api.html
