---
title:  "Cookie&Session"
excerpt: Cookie와 Session 기능에 관한 공부
categories:
  - Servlet
---

### 쿠키 API
- Javax.servlet.http.Cookie 사용
- HttpServletResponse의 addCookie() 메서드 이용해 클라이언트 브라우저에 쿠키 전송
- HttpServletRequest의 getCookie() 메서드 이용해 쿠키를 서버로 가져옴
- Cookie의 setMaxAge()메서드 이용해 유효시간 -1로 설정하면 세션쿠키 생성됨

### 세션
- 정보가 서버의 메모리에 저장됨 
- 쿠키보다 보안에 유리함 
- 서버에 부하 줄 수 있음 
- 브라우저(사용자)당 한 개의 세션(세션ID)이 생성됨 
- 브라우저의 세션 연동은 세션 쿠키를 이용함 
- 세션은 유효 시간을 가짐(기본 유효 시간은 30분) 
- 로그인 상태 유지기능이나 쇼핑몰의 장바구니 담기 기능 등에 주로 사용됨 

### 세션 API
- 서블릿에서 HttpSession 클래스 객체를 생성해서 사용함 
- HttpServletRequest의 getSession() 메서드를 호출해서 얻음 
- getSession(), getSession(true) : 기존 세션객체가 존재하면 반환하고, 없으면 새로 생성 
- getSession(false) : 기존 세션객체가 존재하면 반환하고, 없으면 null 반환 

### 세션에 바인딩 
- \<Manager pathname=**/\> 주석해제 
- Session.setAttribute("속성명", 객체); 
- 브라우저에서 쿠키사용 금지하면 encodeURL() 사용하여 세션 사용함 
- Url로 JsessionID를 전송하여 세션기능 수행함 
