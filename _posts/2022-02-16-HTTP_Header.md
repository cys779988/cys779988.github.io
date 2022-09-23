---
title:  "HTTP Header"
excerpt: HTTP 헤더
categories:
  - etc
---

## 공통헤더
- 요청과 응답에 모두 사용되는 헤더

#### Date
  
```
Date: Thu, 10 Feb 2022 10:27:30 GMT
```  

- Http 메시지가 만들어진 시각. 자동으로 만들어진다.

#### Connection
  
```
Connection: keep-alive
```  

- HTTP/1.1에서는 기본적으로 keep-alive, HTTP/2에서는 사라진다.

#### Content-Length
  
```
Content-Length: 52
```  

- 요청과 응답 메시지의 본문 크기를 바이트 단위로 표시함. 메시지 크기에 따라 자동으로 만들어진다.

#### Content-Type
  
```
Content-Type: text/html; charset=utf-8
```  

- 컨텐츠의 타입(MIME)과 문자열 인코딩(uft-8 등)을 명시. Accept 헤더, Accept-Charset 헤더와 대응된다.
- 메시지 내용이 text/html, 문자열은 utf-8임을 알려준다.
- 프론트엔드에서 서버로 데이터를 보낼 때는 Content-Type이 www-url-form-encoded나 multipart/form-data 등.

#### Content-Language
- 사용자의 언어를 뜻함. 요청이나 응답이 어떤 언어인지와는 관련 없이 페이지 언어가 영어라도 Content-Language는 ko-KR일 수 있다.


#### Content-Encoding
  
```
Content-Encoding: gzip, deflate
```  

- 컨텐츠가 압축된 방식. 응답 컨텐츠를 br, gzip, deflate 등의 알고리즘으로 압축해서 보내면, 브라우저가 그에 맞게 압축해제하여 사용한다.
- 컨텐츠 용량이 줄어들기 때문에 압축을 권장함. 요청이나 응답의 전송 속도도 빨라지고, 데이터 소모량도 줄어든다.

## 요청헤더

#### Host
  
```
Host: localhost:8080
```  

- 서버의 도메인 네임이 나타나는 부분
- Host 헤더는 반드시 하나가 존재해야한다.

#### User-Agent
  
```
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/98.0.4758.82 Safari/537.36
```  

- 현재 사용자가 어떤 클라이언트를 이용해 요청을 보냈는지 알려준다. 보통 User-Agent 헤더를 활용하여 브라우저나 디바이스별 접속자 통계를 냄

#### Accept
  
```
Accept: text/html
Accept: image/png, image/gif
Accept: text/*
```  

- 요청을 보낼 때 서버에 타입(MIME)의 데이터를 보내줬으면 좋겠다고 명시할 때 사용된다.
- 콤마로 여러타입을 동시에 적어줄 수 있고, 와일드카드 문자로 표현할 수도 있다.
- Accept는 Accept-Encoding, Accept-Charset, Accept-Language 등이 있는데 공통 헤더의 Content와 대응된다.
- Accept로 원하는 형식을 보내면, 서버에서 그에 맞게 보내주면서 응답 헤더의 Content를 알맞게 설정.

#### Authorization
  
```
Authorization: Bearer XXXXXXXXXXXXX
```  

- 인증토큰(JWT, Bearer)을 서버로 보낼 때 사용하는 헤더. API 요청을 할 때 토큰이 없으면 거절당하는데 이 때 Authorization을 사용.
- 보통의 경우 Basic이나 Bearer같은 토큰의 종류를 먼저 알리고 그 다음 실제 토큰 문자를 적어 보낸다.

#### Origin
- POST 같은 요청을 보낼 때, 요청이 어느 주소에서 시작되었는지를 나타낸다. 요청을 보낸 주소와 받는 주소가 다르면 CORS 문제가 발생할 수 있다.

#### Referer
  
```
Referer: localhost:8080/main
```  

- 현재 페이지 이전의 페이지 주소가 담겨 있으며 어떤 페이지에서 현재 페이지로 들어왔는지 알 수 있다.

## 응답헤더

#### Access-Control-Allow-Origin
- 요청을 보내는 프론트엔드 주소와 받는 백엔드 주소가 다르면 CORS 에러가 발생하는데 이때 서버에서 응답 메시지 Access-Control-Allow-Origin 헤더에 프론트엔드 주소를 적어주면 에러가 발생하지 않는다.
- 프로토콜, 서브도메인, 도메인, 포트 중 하나만 달라도 CORS 에러가 발생하는데 주소를 지정하기 싫다면 보안에 취약해지지만 와일드카드 문자로 모든 주소에 CORS 요청을 허용하면 된다.
- CORS 요청 시에는 미리 OPTIONS 주소로 서버가 CORS를 허용하는지 물어보고 이 때 Access-Control-Request-Method로 실제로 보내고자 하는 메서드를 알린 후 Access-Control-Request-Headers로 실제로 보내고자 하는 헤더들을 알린다.

#### Allow
- Allow 헤더들은 Request에 대응되는데 서버가 허용하는 메서드와 헤더를 응답하는데 사용된다.
- Request와 Allow가 일치하면 CORS 요청이 이루어진다.
- Allow 헤더는 Access-Control-Allow-Methods랑 비슷하지만, CORS 요청 외에도 적용된다는 데에 차이가 있다. GET localhost:8080/scrap은 되고, POST localhost:8080/scrap은 허용하지 않는 경우, 405 Method Not Allowed 에러를 응답하면서 헤더로   ```Allow: GET```  를 보내면 된다.

#### Content-Disposition
  
```
Content-Disposition: inline
Content-Disposition: attachment; filename='filename.png'
```  

- 응답 본문을 브라우저가 어떻게 표시해야 할지 알려주는 헤더로. inline인 경우 웹페이지 화면에 표시되고 attachment인 경우 다운로드된다.

#### Location
  
```
HTTP/1.1 302 Found
Location: /
```  

- 300번대 응답이나 201 Created 응답일 때 어느 페이지로 이동할지 알려주는 헤더.
- Location 값의 주소로 리다이렉트함.

#### Content-Security-Policy
- 다른 외부 파일들을 불러오는 경우, 차단할 소스와 불러올 소스를 여기에 명시할 수 있다.
- 하나의 웹 페이지는 다양한 외부 소스들을 불러오는데 해커들에 의해 원하지 않는 파일을 불러오게 될 경우 Content-Security-Policy로 허용할 외부 소스만 지정할 수 있다.

  
```
Content-Security-Policy: default-src 'self'
Content-Security-Policy: default-src https:
Content-Security-Policy: default-src 'none'
```  

- self로 지정하면 자신의 도메인의 파일들만 가져올 수 있고 https로 지정하면 https를 통해서만 파일을 가져오며 none으로 지정하면 파일을 가져올 수 없게 된다.
- default-src는 모든 외부 소스에 대해 적용되는 것이고, 각각 따로 지정할 수도 있다.

  
```
Content-Security-Policy: font-src 'self'; script-src localhost:8080 'unsafe-inline';
img-src 'self'; style-src 'self' 'unsafe-inline'; object-src 'none'
```  

- font-src, script-src, img-src, style-src, object-src 등이 있고, 소스 옵션으로는 도메인이나, https:, 'unsafe-inline'(인라인 태그 허용), 'unsafe-eval'(eval 함수 허용) 등이 있다.

