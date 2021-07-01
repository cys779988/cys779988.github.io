---
title:  "nodejs hash모듈"
excerpt: nodejs hash모듈
categories:
  - nodejs
---

> ## hash모듈 SHA256

```
yarn add sha256
//npm install sha256
```  

- server/config/hashing.js 생성

```javascript
// server/config/hashing.js

module.exports ={
    enc : function(id, pwd, salt){
        const sha256 = require('sha256');
        return sha256(id + pwd+ salt)
    },

}
```  

```javascript
// server/controller.js


const hashing = require(path.join(__dirname, 'config', 'hashing.js'))

...
```  

- salt 는 hash를 할 때 들어가는 인자값에 고정된 어떠한 문자열을 추가하는 과정
- 설정 디렉토리 안에 문자열로 salt 값 지정

```
//db.json

...

, "salt" : "agf1a3g4fy31fh2ad31t45621hfd21ga547yfgsd43d2gsda4feaw"

```  

- controller.js 에서 salt 를 불러옴

```javascript
const salt = require(path.join(__dirname, 'config', 'db.json'))
    .salt
```  

- password에 대한 hash 값 구하기

```javascript
// server/controller.js

        sendPw : (req, res) => {
            const body = req.body;
            const hash = hashing.enc(body.id, body.password, salt)
        },
```  

### DB에 정보 INSERT 하기

- models/admin.js 생성

```javascript
module.exports = (sequelize, DataTypes) => {
    return sequelize.define(
        'admin',
        {
            id: {
                type: DataTypes.INTEGER,
                primaryKey: true,
                autoIncrement: true
            },

            user_id: {
                type: DataTypes.STRING(30),
                allowNull: false,
                unique: true
            },

            password: {
                type: DataTypes.STRING(100),
                allowNull: false
            },
        },
        {
            charset: 'utf8',
            collate: 'utf8_general_ci',
            timestamps: false,
        }
    )

}
```  

- DB에 테이블 생성되도록 server/models/index.js에 코드 추가

```
db.Admin = require('./admin')(sequelize, Sequelize);
```  
