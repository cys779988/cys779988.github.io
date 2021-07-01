---
title:  "React + SpringFramework 연동"
excerpt: react
categories:
  - react
---

## 초기 환경설정
- Global Dependency 설치
  
  ```
  sudo npm install -g webpack webpack-dev-server
  ```  
- cmd에서 cd를 사용해 개발하고자 하는 workspace로 이동함
- webpack : 브라우저 위에서 import 및 require로 불러올 수 있게 해주고, 자바스크립트 파일을 하나로 합쳐주는 역할
- webpack-dev-server : 별도의 서버를 구축하지 않고도 static 파일을 다루는 웹서버를 열 수 있으며, hot-loader를 통하여 코드가 수정될 때마다 자동으로 reload 되게 해 줌

## 프로젝트 생성
  
  ```
  mkdir react
  cd react
  npm init
  ```  
- npm init 명령어를 사용하여 프로젝트에 대한 여러 정보를 입력할 수 있게 하고, 그 정보를 기반으로 기본 package.json 파일을 생성함

## Dependency 및 Plugin 설치
- react 설치
  
  ```
  npm install --save react react-dom
  ```  

- 개발에 필요한 의존 모듈 설치
  
  ```
  npm install --save-dev react-hot-loader webpack webpack-dev-server
  npm install --save-dev babel-core babel-loader babel-preset-es2015 babel-preset-react
  ```  
  
- redux 모듈 설치
  
  ```
  npm install --save redux react-redux
  ```  

- immutability helper 사용을 위한 모듈 설치
  
  ```
  npm install --save react-addons-update
  ```  
  
## 디렉토리 설정
  
  ```
  react 
      package.json
      public                            # 서버 public path
                  index.html            # 메인 페이지
      src                               # React.js 프로젝트 루트
                  components            # 컴포넌트 폴더
                            App.js      # App 컴포넌트
                  index.js              # Webpack Entry point
      webpack.config.js                 # Webpack 설정파일(직접 만들어야함)
  ```  
  
## 모듈 파일 디렉토리 생성
- webpack 을 이용한 Front-End 개발시에는 모듈들을 모아두는 디렉토리가 필요
- 모듈 파일을 빌드시 포함되지 않는 위치인 src/main에 front폴더 생성

## webpack 설정
- global로 설치한 webpack을 project level(위에서 front 폴더 만든 src/main)에 한번 더 설치
- 추가로 --save 옵션을 주어 package.json 파일에 자동으로 dependency를 추가해줌
  
```
npm install --save-dev webpack
```  

## webpack.config.js
  
```javascript
const path = require('path');

module.exports = {
  context: path.resolve(__dirname, 'front'),
  entry: {
    home: './home.js',
  },
  output: {
    path: path.resolve(__dirname, 'webapp/resources'),
    filename: '[name].js',
    publicPath: '/proejctName/resources',
  }
};

```  
- context 설정 : 앞서 생성한 모듈 파일의 디렉토리로 설정
- path 설정 : 엔트리 파일(번들 파일)이 저장 될 위치를 설정. 이 디렉토리는 빌드시 포함되는 디렉토리이며, WAS를 통해 접근이 가능해야함
- publicPath 설정 : 각종 리소스를 URL로 통해 접근시 URL 앞에 붙는 공통 경로. 따라서 WAS의 root context와 리소스가 저장되는 디렉토리르 합쳐서 설정

## Spring Context 설정
- 엔트리 파일(번들 파일)이 저장되는 디렉토리는 WAS를 통해 접근 가능해야함. 따라서 Spring Context 파일 설정
  
```
<resources mapping="/resources/**" location="/resources/" />
```  

## package.json 파일 설정
- 해당 스크립트들은 npm run 스크립트명 으로 실행이 가능
- w(watch) 옵션을 주어서 모듈 파일이 변경시 바로바로 번들링 되게끔 함
  
```
"scripts": {
  "develop": "webpack -w --mode development --devtool inline-source-map",
  "build": "webpack --mode production"
  },
```  

## Spring 프로젝트에 webpack 적용
- Spring 프로젝트는 개발 후 배포시 보통 war 파일로 배포됨.  
- 그리고 webpack 을 이용해서 Front-End 개발을 하면 모듈 파일과, 번들 파일 두 종류의 파일이 생성.  
- 모듈 파일은 개발시에만 필요. 어차피 번들링하면 번들 파일에 모듈이 모두 포함되기 때문. 따라서 배포시 모듈 파일을 괜히 war 파일에 포함시켜 war 파일의 용량을 키울 필요도 없음.
