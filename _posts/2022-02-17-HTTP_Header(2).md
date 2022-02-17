---
title:  "HTTP Header"
excerpt: HTTP 쿠키&캐시 헤더, X헤더
categories:
  - http
---

## 캐시
- 여기서 말하는 캐시는 개인캐시를 뜻한다. CDN같은 공유 캐시가 아니라 브라우저에 응답으로 온 HTML이나 JSON같은 데이터가 저장되어 나중에 서버에 요청을 보내지 않고도 브라우저에 저장된 응답을 사용할 수 있다.
- 보통 캐싱은 GET 요청에만 사용한다. GET은 REST적 의미로 데이터를 가져오는 것을 의미하는데 가져온 데이터를 보관해두고 사용하기 위해서이다.

#### Cache-Control

  
```
Cache-Control: no-store
```  

- 아무것도 캐싱하지 않는 옵션

  
```
Cache-Control: no-cache
```  

- 캐시를 사용하기 전에 서버에 캐시를 사용해도 되는 물어보는 옵션

  
```
Cache-Control: must-revalidate
```  

- 만료된 캐시만 서버에 확인을 받도록 하는 옵션

  
```
Cache-Control: public 또는 private
```  

- public은 공유캐시에 저장해도 된다는 뜻이고, private은 브라우저 같은 특정 사용자환경에만 저장하라는 뜻

  
```
Cache-Control: public, max-age=3600
```  

- max-age로 캐시 유효시간을 설정. 초 단위이므로   ```max-age=3600```   옵션은 1시간.
- 유효시간이 지나면 응답 캐시는 만료된 것으로 여겨진다.

  
  
위 옵션들은   ```no-store, no-cache, must-revalidate```   와 같이 콤마로 구분해서 혼합하여 사용할 수 있다.
Cache-Control는 응답헤더 말고 요청헤더로 사용하여 서버에 있는 캐시를 가져오지 않도록 요청시 Cache-Control 헤더에 넣어줄 수 있다.

#### Age

  
```
Age: 60
```  

- 캐시 응답을 나타내는데, max-age 시간 내에서 얼마만큼의 시간이 지났는지 초 단위로 알려준다. 1분이 지난 경우 60.

#### Expires

  
```
Expires: Thu, 10 Feb 2022 10:27:30 GMT
```  

- Cache-Control과 별개로 응답에 Expires라는 헤더를 주면 응답 컨텐츠가 언제 만료되었는지 표기할 수 있다. Cache-Control의 max-age가 있는 경우 이 헤더는 무시된다.

#### ETag

  
```
ETag: W/"15420-1644196791393"
```  

- HTTP 컨텐츠가 바뀌었는지 식별할 수 있는 태그.
- 같은 URL의 리소스가 변경된다면, 새로운 ETag가 생성된다. ETag는 지문과 같은 역할을 하면서 다른 서버들이 추적하는 용도에 이용되기도 한다.
- ETag가 바뀌어지면 서버가 클라이언트의 응답 내용이 달라진 것을 인지하고 현재 캐시를 지우고 새로 컨텐츠를 내려받을 수 있게된다. 서버에서 무기한으로 지속될 수 있드록 설정할 수도 있다.

#### If-None-Match

  
```
If-None-Match: W/"3bf2-wdj3VvN8/CvXVgafkI30/TyczHk"
```  

- 서버에 ETag가 달라졌는지 검사하여 ETag가 다를 경우에만 새 컨텐츠를 전해달라는 옵션
- 만약 ETag가 같다면 서버는 304 Not Modified를 응답해서 캐시를 그대로 사용하게 한다.
