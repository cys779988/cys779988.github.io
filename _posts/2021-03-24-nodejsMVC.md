---
title:  "nodejs MVC패턴"
excerpt: nodejs MVC패턴
categories:
  - nodejs
---

> ## MVC 패턴이 필요한 이유

#### API로 접근하는 라우터
#### API를 처리하는 Controller
#### 데이터를 관리하는 Model

- 각각의 역할들에 대해 체계적으로 관리할 수 있고 코드의 가독성을 높힘


> ## MVC 패턴 적용

### Router 설정

- server 디렉토리 안에 route.js 생성, 아래 코드 추가

```javascript
const express = require('express');
const router = express.Router();

router.get('/get/data');

router.post('/add/data');
router.post('/modify/data');
router.post('/delete/data');

module.exports = router;
```  

- router 라는 변수를 통해서 API 주소를 연결
-   ```router.HTTP 메소드(API주소) ```  형태로 사용
- GET 방식이면   ```router.get```  , POST 방식이면   ```router.post```  
- 서버에서 route.js에 접근할 수 있도록 server.js에 아래 코드 추가

```javascript
const router = require('./route');
(...)
app.use('/', router);
```  

- router 변수에 route.js 를 불러온 뒤   ```app.use```   을 이용해서 라우터를 연결하는 코드


### Controller 설정

- server 디렉토리 안에 controller.js 생성

```javascript
//controller.js

const path = require('path');
const model = require('./model');

module.exports = {
    needs: () => upload,
    api : {
        getData : (req, res) => {
            model.api.getData( data => {
                console.log('연결성공')
                return res.send(data)
            })
        },
        addData : (req, res) => {

        },
        modifyData : (req, res) => {

        },
        deleteData : (req, res) => {

        },
    }
}
```  

### Model 설정

- server 디렉토리 안에 model.js 생성

```javascript
const sequelize = require('./models').sequelize;

const {
    Teacher,
    Sequelize: {Op}
} = require('./models');
sequelize.query('SET NAMES utf8;');

module.exports = {
    api : {
        getData : callback => {
            Teacher.findAll()
            .then( result => { callback(result) })
            .catch( err => { throw err })
        },
    }
}
```  

- server.js 에 있던 코드를 server/model.js로 옮김
- 이제 model.js 에서 DB Table에 대한 정보들을 담고 있고 Sequelize를 이용해서 데이터에 접근할 수 있음
- Controller와 동일하게 key, value 값을 가지는 객체에 Controller가 접근 하게 되면 callback 함수를 이용해 해당 테이블을 조회한 값을 다시 Controller에게 넘겨줌
- Controller에서 모델을 불러올 수 있는 코드 작성

```javascript
const path = require('path');
const model = require('./model');

module.exports = {
    needs: () => upload,
    api : {
        getData : (req, res) => {
            model.api.getData( data => {
                console.log('연결성공')
                return res.send(data)
            })
        },
        addData : (req, res) => {

        },
        modifyData : (req, res) => {

        },
        deleteData : (req, res) => {

        },
    }
}
```  

- model 변수에 server/model.js의 정보를 담고   ```model.api.getData```  로 Model을 실행함
- 이 때 인자로 보내는 'data'는 Model로 보내는 인자이고 Model에서 조회한 데이터는 이 인자에 담겨져 callback함수로 인해 다시 컨트롤러로 전송

### Clinet -> Router -> Controller -> Model(DB) -> Controller -> Client
