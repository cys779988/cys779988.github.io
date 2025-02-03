---
title:  "HTTP Header"
excerpt: HTTP 쿠키&캐시 헤더, X헤더
categories:
  - etc
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

  
## HTTP 쿠키
- 쿠키는 브라우저에 저장되는 작은 데이터 조각으로 임시 데이터 보관 또는 웹페이지 자동화 등에 사용된다.
- 두 요청이 동일한 브라우저에서 들어왔는지 아닌지를 판단할 때 주로 사용된다. 이를 통해 사용자의 로그인 상태를 유지할 수 있다.
- 세션관리, 개인화(사용자선호), 트래킹(사용자 행동 기록/분석) 등에 사용된다.


#### Set-Cookie
- HTTP 요청을 수신할 때, 서버는 응답과 함께 Set-Cookie 헤더를 전송할 수 있다.
- 쿠키는 보통 브라우저에 의해 저장되며, 그 후 쿠키는 같은 서버에 의해 만들어진 요청(Request)들의 HTTP 헤더안에 포함되어 전송된다.
- 만료일 혹은 지속시간(duration)도 명시될 수 있고, 만료된 쿠키는 더 이상 보내지지 않는다. 추가적으로 특정 도메인 혹은 경로 제한을 설정할 수 있으며 이는 쿠키가 보내지는 것을 제한할 수 있다.

  
```
Set-Cookie: <cookie-name>=<cookie-value>; option
```  

-   ```<cookie-name>```   은 제어 문자 및 공백, 탭(\t)를 제외한 아스키 코드 문자로 구성되어야 한다. 또한,   ```( ) < > @ , ; : \ " /  [ ] ? = { }```   같은 문자가 포함될 수 없다.
-   ```<cookie-value>```  는 필요하다면 쌍 따옴표로 묶여질 수 있고 아스키 코드 문자로 구성되어야 하며,   ```( ) < > @ , ; : \ " /  [ ] ? = { }```   같은 문자가 포함될 수 없다.
-   ```__Secure-prefix```  는   ```__Secure-prefix```  로 시작되는 쿠키이름은 반드시 secure 플래그가 설정되어야 하고, HTTPS 여야 한다.
-   ```__Host-prefix```  는   ```__Host-prefix```  로 시작되는 쿠키들은 secure 플래그가 설정되어야 하고, 마찬가지로 HTTPS 여야 하고, 도메인이 지정되지 않아야한다.(따라서 서브 도메인에 쿠키를 공유할 수 없다.) 그리고 경로는 반드시 "/"여야 한다.



- Session cookie
  
```
Set-Cookie: sessionid=38afes7a8; HttpOnly; Path=/
```  

- Permanent cookie
  
```
Set-Cookie: id=a3fWa; Expires=Thu, 21 Oct 2021 07:28:00 GMT; Secure; HttpOnly
```  

- Invalid domains
  
```
Set-Cookie: qwerty=219ffwef9w0f; Domain=somecompany.co.uk; Path=/; Expires=Wed, 30 Aug 2019 00:00:00 GMT
```  

- Cookie prefixes
  
```
// Both accepted when from a secure origin (HTTPS)
Set-Cookie: __Secure-ID=123; Secure; Domain=example.com
Set-Cookie: __Host-ID=123; Secure; Path=/

// Rejected due to missing Secure directive
Set-Cookie: __Secure-id=1

// Rejected due to the missing Path=/ directive
Set-Cookie: __Host-id=1; Secure

// Rejected due to setting a domain
Set-Cookie: __Host-id=1; Secure; Path=/; domain=example.com
```  

#### 쿠키의 라이프타임
- 세션 쿠키는 현재 세션이 끝날 때 삭제된다. 브라우저는 현재 세션이 끝나는 시점을 정의하며, 어떤 브라우저들은 재시작할 때 세션을 복원해 세션 쿠키가 무기한 존재할 수 있도록 한다.
- 영속적인 쿠키는 Expires 속성에 명시된 날짜에 삭제되거나, max-age 속성에 명시된 기간 이후에 삭제된다.

  
```
Expires=<date>
Max-Age=<number>
```  

- Expires 와 Max-Age가 같이 지정되었을 때 Max-Age 값을 더 우선시합니다.

#### Secure과 HttpOnly 쿠키
- Secure 쿠키는 HTTPS에서만 전송된다. HTTP는 쿠키에 Secure 설정을 지시할 수 없다.
- HttpOnly 쿠키는 XSS 공격을 방지하기 위해 설정하며, 자바스크립트의   ```document.cookie```  API에 접근할 수 없다. 서버쪽에서 지속되고 있는 세션의 쿠키는 자바스크립트를 사용할 필요성이 없기 때문에 HttpOnly 플래그가 설정된다.

#### 쿠키의 스코프

- Domain, Path 디렉티브는 어떤 URL을 쿠키가 보내야 하는지 쿠키의 스코프를 정의한다.

  
```
Domain=<domain-value>
```  

- Domain은 쿠키가 전송되게 할 호스트들을 명시한다. 만약 명시되지 않는다면 현재 문서 위치의 호스트 일부를 기본값으로 한다. 도메인이 명시되면, 서브도메인들은 항상 포함된다.
- 만약   ```Domain=mozila.org```  가 설정되면, 쿠키들은   ```developer.mozilla.org```  와 같은 서브도메인 상에 포함되게 된다.

  
```
Path=<path-value>
```  

- Path는 Cookie 헤더를 전송하기 위하여 요청되는 URL 내에 반드시 존재해야 하는 URL 경로. %x2F ("/") 문자는 디렉티브 구분자로 해석된다.
-   ```Path=/docs```  로 설정되면   ```/docs, /docs/Web, /docs/Web/HTTP```  가 매치된다.

#### SameSite 쿠키

  
```
SameSite=Strict
SameSite=Lax
```  

- 쿠키가 cross-site 요청과 함께 전송되지 않았음을 요구하게 만들어, cross-site 요청 위조 공격(CSRF)에 대해 보호방법을 제공한다. SameSite 쿠키는 아직 실험중에 있으며 브라우저에서 제공되지 않는다.


#### Cookie
- Cookie HTTP 요청 헤더는   ```Set-Cookie```   헤더와 함께 서버에 의해 이전에 전송되어 저장된 HTTP cookies를 포함한다.
- Cookie 헤더는 선택적이고, 만약 브라우저의 사생활 보호 설정이 쿠키를 block 할 경우 생략될 수도 있다.

  
```
Cookie: <cookie-list>
Cookie: name=value
Cookie: name=value; name2=value2; name3=value3
```  

#### 세션 하이재킹과 XSS
- 쿠키는 대개 웹 애플리케이션에서 사용자와 그들의 인증된 세션을 식별하기 위해 사용된다.
- 쿠키를 가로채는 것은 인증된 사용자의 세션 하이재킹으로 이어질 수 있다. 쿠키를 가로채는 일반적인 방법은 소셜 공학 사용 혹은 애플리케이션 내 XSS 취약점을 이용하는 것을 포함한다.
- HttpOnly 쿠키 속성은 자바스크립트를 통해 쿠키 값에 접근하는 것을 막아 이런 공격을 막는데 도움을 줄 수 있다.


#### 서드파티 쿠키
- 쿠키는 그와 관련된 도메인을 가진다. 이 도메인이 당신이 현재 보고 있는 페이지의 도메인과 동일하다면, 그 쿠키는 퍼스트파티 쿠키라고 불린다.
- 만약 도메인이 다르다면, 서드파티 쿠키라고 불린다.
- 퍼스트파티 쿠키가 그것을 설정한 서버에만 전송되는데 반해, 웹 페이지는 다른 도메인의 서버 상에 저장된(광고 배너 등) 이미지 혹은 컴포넌트를 포함할 수도 있다.
- 이러한 서드파티 컴포넌트를 통해 전송되는 쿠키들을 서드파티 쿠키라고 부르며 웹을 통한 광고와 트래킹에 주로 사용된다. 대부분의 브라우저들은 기본적으로 서드파티 쿠키를 따르지만, 그것을 차단하는데 이용되는 애드온들이 있다.

