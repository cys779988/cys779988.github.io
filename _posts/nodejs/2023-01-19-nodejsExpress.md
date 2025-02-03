---
title:  "Node.js Express 기초"
excerpt: Node.js Express 기초
categories:
  - nodejs
---

**Express**는 웹 및 모바일 애플리케이션을 위한 강력한 기능을 제공하는 Node.js 웹 애플리케이션 프레임워크다. 수많은 HTTP 유틸리티 메서드 및 미들웨어를 통해 쉽고 빠르게 API를 작성할 수 있다.

## 환경설정
  
```
npm init

// --save 옵션을 통해 설치된 노드 모듈은 package.json 파일 내의 dependencies 목록에 추가된다. 이후 앱 디렉토리에서 npm install을 실행하면 종속 항목 목록 내의 모듈이 자동으로 설치된다.
npm install express --save

// Express 애플리케이션 생성기 설치로 애플리케이션의 골격을 신속하게 작성할 수 있다.
npm install express-generator -g

express --view=pug myapp

cd myapp
npm install

// MacOS or Linux
DEBUG=myapp:* npm start

// WINDOW
set DEBUG=myapp:* & npm start
```  

## 라우팅
**라우팅**은 URI 및 특정한 HTTP 요청 메서드인 특정 엔드 포인트에 대한 클라이언트 요청에 애플리케이션이 응답하는 방법을 결정하는 것을 말한다.  
각 라우트는 하나 이상의 핸들러 함수를 가질 수 있으며, 이러한 함수는 라우트가 일치할 때 실행된다.

#### 라우트 정의
  
```js
const express = require('express');
const app = express()

app.METHOD(PATH, HANDLER)
```  

- app : express 인스턴스
- METHOD : HTTP 요청 메서드
- PATH : 서버에서의 경로
- HANDLER : 라우트가 일치할 때 실행되는 함수

#### 라우트 메서드
- 라우트 메서드는 HTTP 메서드 중 하나로부터 파생되며, express 클래스의 인스턴스에 연결된다.
- Express에서 지원하는 라우팅 메서드는 다음과 같다.

  
```
get, post, put, head, delete, options, trace, copy, lock, mkcol, move, purge, propfind, proppatch, unlock, report, mkactivity, checkout, merge, m-search, notify, subscribe, unsubscribe, patch, search 및 connect ...
```  

- 특수한 라우팅 메서드인 `app.all()`은 어떠한 HTTP 메서드로부터도 파생되지 않는다. 이 메서드는 모든 요청 메서드에 대해 한 경로에서 미들웨어 함수를 로드하는 데 사용된다.


```js
app.all('/secret', function(req, res, next)) {
    console.log('Accessing the secret section ...');
    next();
}
```  

#### 라우트 경로
라우트 경로는 요청 메서드와의 조합을 통해 요청이 이루어질 수 있는 엔드 포인트를 정의한다. 문자열, 문자열 패턴 또는 정규식일 수도 있다.

  
```js
// about
app.get('/about', function (req, res) {
    res.send('about');
});

// acd, abcd
app.get('/ab?cd', function (req, res) {
    res.send('ab?cd');
});

// 라우트 이름에 'a'가 포함된 모든 항목
app.get(/a/, function (req, res) {
    res.send('/a/');
});

// butterfly, dragonfly
app.get(/.*fly$/, function (req, res) {
    res.send('/.*fly$/');
});

```  

#### 라우트 핸들러
미들웨어와 비슷하게 동작하는 여러 콜백 함수를 제공하여 요청을 처리할 수 있다. 유일한 차이점은 이러한 콜백은 `next('route')`를 호출하여 나머지 라우트 콜백을 우회할 수도 있다는 점이다. 이러한 메커니즘을 이용하면 라우트에 대한 사전 조건을 지정한 후, 현재의 라우트를 계속할 이유가 없는 경우에는 제어를 후속 라우트에 전달할 수 있다.

  
```js
// 2개 이상의 콜백함수로 라우트 처리
app.get('/example/b', function (req, res, next) {
    console.log('the response will be sent by the next function ...');
    next();
}, function (req, res) {
    res.send('Hello from B!');
});


// 콜백 함수 배열로 라우트 처리
var cb0 = function (req, res, next) {
    console.log('CB0');
    next();
}

var cb1 = function (req, res, next) {
    console.log('CB1');
    next();
}

var cb2 = function (req, res) {
    res.send('Hello from C!');
}

app.get('/example/c', [cb0, cb1, cb2]);
```  

#### 응답 메서드
응답 오브젝트에 대한 메서드는 응답을 클라이언트로 전송하고 요청-응답 주기를 종료할 수 있다. 라우트 핸들러로부터 다음 메서드 중 어느 하나도 호출되지 않는 경우, 클라이언트 요청은 정지된 채로 방치된다.

  
메서드 | 설명
---- | ----
res.download() | 파일이 다운로드되도록 프롬프트한다.
res.end() | 응답 프로세스를 종료한다.
res.json() | JSON 응답을 전송한다.
res.jsonp() | JSONP 지원을 통해 JSON 응답을 전송한다.
res.redirect() | 요청의 경로를 재지정한다.
res.render() | 보기 템플릿을 렌더링한다.
res.send() | 다양한 유형을 응답을 전송한다.
res.sendFile() | 파일을 Octet 스트림 형태로 전송한다.
res.sendStatus() | 응답 상태 코드를 설정한 후 해당 코드를 문자열로 표현한 내용을 본문으로 전송한다.
  

#### app.route()
`app.route()`를 이용하면 라우트 경로에 대하여 체인 가능한 라우트 핸들러를 작성할 수 있다. 경로는 한 곳에 지정되어 있으므로, 모듈식 라우트를 작성하면 중복과 실수를 줄일 수 있다.

  
```js
app.route('/book')
    .get(function(req, res) {
        res.send('Get a random book');
    })
    .post(function(req, res) {
        res.send('Add a book');
    })
    .put(function(req, res) {
        res.send('Update the book');
    });
```  

#### express.Router
`express.Router` 클래스를 사용하면 모듈식 마운팅 가능한 핸들러를 작성할 수 있다. Router 인스턴스는 완전한 미들웨어이자 라우팅 시스템이며, 미니 앱이라고 불린다.

  
```js
// users.js
var express = require('express');
var router = express.Router();

router.use(function timeLog(req, res, next) {
    console.log('Time: ', Date.now());
    next();
});

router.param('id', function(req, res, next, id) {
  console.log(`id : ${id}`);
  next();
})

router.get('/', function(req, res) {
    res.send('root');
});

router.get('/:id', function(req, res) {
  res.send(req.params.id);
});

router.get('/about', function(req, res) {
    res.send('about');
});

module.exports = router;
```  

  
```js
// app.js
var usersRouter = require('./routes/users');
...
app.use('/users', usersRouter);
```  

## 미들웨어 사용
Express는 자체적인 최소한의 기능을 갖춘 라우팅 및 미들웨어 웹 프레임워크이며, Express 애플리케이션은 기본적으로 일련의 미들웨어 함수 호출이다.  

미들웨어 함수는 요청(`req`) 및 응답(`res`) 오브젝트, 요청-응답 주기 중 그 다음 미들웨어 함수에 대한 액세스 권한을 갖는 함수다. 그 다음 미들웨어 함수는 일반적으로 `next` 변수로 표시한다.

#### 미들웨어 함수의 수행 작업
- 모든 코드를 실행.
- 요청 및 응답 오브젝트에 대한 변경을 실행.
- 요청-응답 주기를 종료.
- 스택 내의 그 다음 미들웨어 함수 호출.

#### 애플리케이션 레벨 미들웨어
`app.use()` 및 `app.METHOD()` 함수를 이용해 미들웨어를 앱 인스턴스에 바인딩하는 미들웨어.

  
```js
// 마운트 경로가 없는 미들웨어 함수. 앱이 요청을 수신할 때마다 실행.
app.use(function (req, res, next) {
    console.log('Time:', Date.now());
    next();
});

// '/user/:id' 경로에 대한 모든 유형의 HTTP 요청에 대해 실행.
app.use('/user/:id', function (req, res, next) {
    console.log('Request Type:', req.method);
    next();
});

// '/user/:id' 경로에 대한 GET 요청을 처리.
app.get('/user/:id', function (req, res, next) {
    res.send('USER');
});
```  

  
```js
app.get('/user/:id', function (req, res, next) {
  console.log('ID:', req.params.id);
  next();
}, function (req, res, next) {
  res.send('User Info');
});

// 호출되지 않는다.
app.get('/user/:id', function (req, res, next) {
  res.end(req.params.id);
});
```  

라우트 핸들러를 이용하면 하나의 경로에 대해 여러 라우트를 정의할 수 있다. 단, 첫 번째 라우트가 요청-응답 주기를 종료시키면 두 번째 라우트는 호출되지 않는다.

  
```js
app.get('/user/:id', function (req, res, next) {
    if (req.params.id == 0) next('route');
    else next();
}, function (req, res, next) {
    res.render('regular');
});

app.get('/user/:id', function (req, res, next) {
    res.render('special');
});
```  

라우터 미들웨어 스택의 나머지 미들웨어 함수들을 건너뛰려면 `next('route')`를 호출하여 제어를 그 다음 라우트로 전달할 수 있다. 단, `next('route')`는 `app.METHOD()` 또는 `router.METHOD()` 함수를 이용해 로드된 미들웨어 함수에서만 동작한다.

#### 라우터 레벨 미들웨어
라우터 레벨 미들웨어는 `express.Router()` 인스턴스에 바인드된다는 점을 제외하면 애플리케이션 레벨 미들웨어와 동일한 방식으로 동작한다.

  
```js
var app = express();
var router = express.Router();

router.use(function (req, res, next) {
    console.log('Time:', Date.now());
    next();
});

router.get('/', function(req, res) {
    res.send('root');
});

router.get('/about', function(req, res) {
    res.send('about');
});

app.use('/', router);
```  

#### 오류 처리 미들웨어
어떠한 함수를 오류 처리 미들웨어 함수로 식별하려면 4개의 인수를 제공해야 한다. `next` 오브젝트를 사용할 필요는 없지만, 시그니처를 유지하기 위해 해당 오브젝트를 지정해야 한다. 그렇지 않으면 `next` 오브젝트는 일반 미들웨어로 해석되어 오류 처리에 실패하게 된다.

  
```js
app.use(function(err, req, res, next) {
    console.error(err.stack);
    res.status(500).send('Something broke!');
})
```  

#### 기본 제공 미들웨어
4.x 버전 이후의 Express는 더 이상 Connect에 종속되지 않는다. `express.static`을 제외하고, 이전에 Express에 포함되었던 미들웨어 함수는 이제 별도의 모듈에 포함되어 있다.

  
```js
var options = {
    dotfiles: 'ignore',
    etag: false,
    extensions: ['htm', 'html'],
    index: false,
    maxAge: '1d',
    redirect: false,
    setHeaders: function (res, path, stat) {
        res.set('x-timestamp', Date.now());
    }
}

// 정적 파일을 제공하기 위해 Express의 기본 제공 미들웨어 함수 express.static 사용
app.use(express.static('public', options));
```  

#### 써드파티 미들웨어
Express 프레임워크 자체적으로 제공하지 않고 따로 설치해야 하는 미들웨어

  
```js
var express = require('express');
var app = express();
var cookieParser = require('cookie-parser');

app.use(cookieParser());
```  



###### Reference

- https://expressjs.com/ko/starter/generator.html

- https://expressjs.com/ko/guide/routing.html

- https://expressjs.com/ko/guide/writing-middleware.html

- https://expressjs.com/ko/guide/using-middleware.html