---
title:  "React + SpringFramework 연동2"
excerpt: react
categories:
  - react
---

## React + SpringFramework 다른방법
  
  ```
  npm i axios
  ```  
- 백엔드와의 통신을 위해서 Axios(엑시오스)를 설치

## App.js 수정
프론트엔드(Create React App)에 src/App.js 파일 수정
  
```javascript
import React, { useState, useEffect } from 'react';
import './App.css';
import customAxios from './customAxios';

function App() {
  // IP주소 변수 선언
  const [ip, setIp] = useState('');

  // IP주소 값을 설정합니다.
  function callback(data) {
    setIp(data);
  }

  // 첫번째 렌더링을 다 마친 후 실행합니다.
  useEffect(
    () => {
      // 클라이언트의 IP주소를 알아내는 백엔드의 함수를 호출합니다.
      customAxios('/ip', callback);
    }, []
  );

  return (
    <div className="App">
      <header className="App-header">
        이 기기의 IP주소는 {ip}입니다.
      </header>
    </div>
  );
}

export default App;
```  

## Axios.js 추가
- 프론트엔드(Create React App)에 src/customAxios.js 파일 추가
- 운영 환경에 배포할 경우에는 15~16행을 주석
  
```javascript
import axios from 'axios'; // 액시오스

export default function customAxios(url, callback) {
  axios(
    {
      url: '/api' + url,
      method: 'post',
      /**
       * 개발 환경에서의 크로스 도메인 이슈를 해결하기 위한 코드로
       * 운영 환경에 배포할 경우에는 15~16행을 주석 처리합니다.
       * 
       * ※크로스 도메인 이슈: 브라우저에서 다른 도메인으로 URL 요청을 하는 경우 나타나는 보안문제
       */
       baseURL: 'http://localhost:8080',
       withCredentials: true,
    }
  ).then(function (response) {
    callback(response.data);
  });
}
```  

## src/main/java 밑에 Controller 및 WebConfig 추가
  - 운영 환경에 배포할 경우에는 15~18행을 주석 처리
  
```javascript
@Configuration
public class WebConfig implements WebMvcConfigurer {
	@Override
	public void addCorsMappings(CorsRegistry registry) {
		registry.addMapping("/api/**").allowCredentials(true).allowedOrigins("http://localhost:3000");
	}
}
```  

## 개발단계에서 프론트엔드 코딩하며 백엔드와 연동
1. (backend 폴더)/target/backend-1.0.0.jar 파일 만듦
2. java -jar ./target/backend-1.0.0.jar 명령 실행하여 백엔드 실행해둠
3. VS Code를 열어놓고 프론트엔드 코딩


## 개발단계에서 백엔드 코딩하며 프론트엔드와 연동
1. npm start로 프론트엔드 실행
2. IDE 열어놓고 백엔드 코딩
