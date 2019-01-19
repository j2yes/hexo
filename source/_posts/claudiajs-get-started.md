---
title: claudiajs-get-started
date: 2019-01-19 22:20:19
tags:
- aws
- lambda
- claudiajs
categories:
- claudiajs
---
### claudiajs : https://claudiajs.com

1. install claudia
2. execute command `npm init`
3. make lambda.js
4. set aws credential on user root path
5. create lambda function as command with profile
6. invoke lambda with profile : `claudia test-lambda --profile claudia`
7. update lambda : `claudia update`
8. mongodb example project

### install claudia

```jshelllanguage
# global installation
npm install claudia -g

# check installation
claudia -version
```

### make lambda project

```jshelllanguage
mkdir example
cd example
npm init
```
### make a lambda.js file : https://docs.aws.amazon.com/lambda/latest/dg/nodejs-prog-model-handler.html

```javascript
// 1.normal example
exports.myHandler = function(event, context) {
   ...
}
// 2.you can use callback
exports.handler = function(event, context, callback) {
    //console.log('Received event:', JSON.stringify(event, null, 2));
    console.log('value1 =', event.key1);
    console.log('value2 =', event.key2);
    console.log('value3 =', event.key3);
    console.log('remaining time =', context.getRemainingTimeInMillis());
    console.log('functionName =', context.functionName);
    console.log('AWSrequestID =', context.awsRequestId);
    console.log('logGroupName =', context.log_group_name);
    console.log('logStreamName =', context.log_stream_name);
    console.log('clientContext =', context.clientContext);
    if (typeof context.identity !== 'undefined') {
        console.log('Cognitoidentity ID =', context.identity.cognitoIdentityId);
    }    
    callback(null, event.key1); // Echo back the first key value
    // or
    // callback("some error type"); 
};
// 3.below is available on node 8.xx
exports.myHandler = async function(event, context) {
   console.log("value1 = " + event.key1);
   console.log("value2 = " + event.key2);  
   return "some success message";
   // or 
   // throw new Error(“some error type”); 
} 
```

### setting AWS IAM 
- https://claudiajs.com/tutorials/installing.html 
- https://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/setting-credentials-node.html

make file `.aws/credentials`  

> first line is profile name.

```jshelllanguage
[claudia]
aws_access_key_id = YOUR_ACCESS_KEY
aws_secret_access_key = YOUR_ACCESS_SECRET
```

### create lambda function

seoul region is `ap-northeast-2`.
 
```jshelllanguage
# create lambda
claudia create --region ap-northeast-2 --handler lambda.myHandler --profile claudia
```

### invoke lambda

```jshelllanguage
#'test-' prefix is required.
claudia test-lambda --profile claudia
```

### update lambda with npm run command

package.json

```json
...
  "scripts": {
    "test": "claudia test-lambda --profile claudia",
    "deploy": "claudia update --profile claudia"
  },
...
```
update command : `npm run deploy`

### example code

[github](https://github.com/j2yes/lambda-mongodb)