---
title:  "Servlet Filter&Listener"
excerpt: Servlet Filter와 Listener 기능에 관한 공부
categories:
  - Servlet
---

## 서블릿 속성
- ServletContext, HttpServlet, HttpServletRequest 객체에 바인딩 되어 저장된 객체(정보)
- 각 서블릿 API의 setAttribute(String name, Object value)로 바인딩함
- 각 서블릿 API의 getAttribute(String name)으로 접근함
- 각 서블릿 API의 removeAttribute(String name)으로 속성을 제거함

## 서블릿 scope
- ServletContext 속성은 애플리케이션 전체에서 접근 가능
- HttpSession 속성은 사용자만 접근 가능
- HttpServletRequest 속성은 해당 요청/응답에 대해서만 접근 가능  
- 각 스코프를 활용해 로그인 상태유지, 장바구니, MVC의 Model과 View의 데이터 전달 기능 구현
  
스코프 종류 | 해당 서블릿 API | 속성의 스코프
---- | ---- | ----
application scope | ServletContext | 애플리케이션 전체에 대해 접근 가능
session scope | HttpSession | 같은 브라우저에서만 접근 가능
request scope | HttpServletRequest | 해당 요청/응답 사이클에서만 접근 가능
  
## URL패턴
서블릿 매핑 시 사용되는 가상의 이름
  
- request.getContextPath() : 컨텍스트 이름만 가져옴  
- request.getRequestURL().toString() : 전체 URL 가져옴  
- request.getServletPath() : 서블릿 매핑 이름만 가져옴  
- request.getRequestURI() : URI를 가져옴

## 필터(Filter)
- 브라우저에서 서블릿에 요청하거나 응답할 때 미리 요청이나 응답과 관련해 여러 가지 작업을 처리하는 기능
- 요청이나 응답 시 공통적인 작업을 처리하는데 이용됨  

## 필터 용도
- 요청 필터
- 사용자 인증 및 권한 검사
- 요청 시 요청 관련 로그 작업
- 인코딩 기능
- 응답 필터
- 응답 결과에 대한 암호화 작업
- 서비스 시간 측정

## 필터관련 API
- javax.servlet.Filter
- javax.servlet.FilterChain
- javax.servlet.FilterConfig

## 필터 매핑 방법
1. 애너테이션 사용 @WebFilter("/*")
2. web.xml에 설정
  
doFilter() 메소드 기준 상단은 요청 필터 기능/ 하단은 응답 필터 기능  

## 서블릿 관련 Listener
서블릿에서 발생하는 이벤트에 대해 처리 할 수 있는 기능  
<img src="https://cys779988.github.io/assets/img/servlet-17.png">  

- HttpSessionBindingListener 이용해 로그인 접속자수 표시  
HttpSessionBindingListener를 구현한 LoginImpl 클래스는 리스너를 따로 등록할 필요 없음  

- HttpSessionListener 이용해 로그인 접속자수 표시  
한 개 아이디로 동시에 접속 가능 횟수를 조절할 수 있음 

## HttpSessionListener 활용한 중복로그인 방지

  
```java
public interface HttpSessionListener extends EventListener {

    public default void sessionCreated(HttpSessionEvent se) {
    }

    public default void sessionDestroyed(HttpSessionEvent se) {
    }
}
```  

- SessionConfig 클래스 생성 후 HttpSessionListener 인터페이스 구현

  
```java
@WebListener
public class SessionConfig implements HttpSessionListener{
	
	private static final Map<String, HttpSession> sessions = new ConcurrentHashMap<String, HttpSession>();
	
	@Override
	public void sessionCreated(HttpSessionEvent se) {
		// TODO Auto-generated method stub
		sessions.put(se.getSession().getId(), se.getSession());
	}
	
	@Override
	public void sessionDestroyed(HttpSessionEvent se) {
		// TODO Auto-generated method stub
		if(sessions.get(se.getSession().getId()) != null) {
			sessions.get(se.getSession().getId()).invalidate();
			sessions.remove(se.getSession().getId());
		}
	}
}

```  

- 로그인 성공시 세션이 생성되면 @WebListener에 의해 세션값이 sessions에 저장
- 로그아웃이나 브라우저를 종료하여 세션이 사라지게 되면 Destroyed에 의해 제거
- 스프링에서 세션은 웹 브라우저가 해당 서버에 접속하면 바로 생성됨
- 생성의 시기가 로그인이 아니라 웹 브라우저 접속시이므로 로그인시 세션값을 추가하는 행위는 update나 set의 개념

  
```java
@WebListener
public class SessionConfig implements HttpSessionListener{
	
	private static final Map<String, HttpSession> sessions = new ConcurrentHashMap<String, HttpSession>();
	
	public synchronized static String getSessionCheck(String type, String compareId) {
		String result = "";
		for (String key : sessions.keySet()) {
			HttpSession value = sessions.get(key);
			if(value != null && value.getAttribute(type) != null && value.getAttribute(type).toString().equals(compareId)) {
				result = key.toString();
			}
		}
		removeSessionForDoubleLogin(result);
		return result;
	}
	
	private static void removeSessionForDoubleLogin(String userId) {
		if(userId != null && userId.length() > 0) {
			sessions.get(userId).invalidate();
			sessions.remove(userId);
		}
	}
	@Override
	public void sessionCreated(HttpSessionEvent se) {
		// TODO Auto-generated method stub
		sessions.put(se.getSession().getId(), se.getSession());
	}
	
	@Override
	public void sessionDestroyed(HttpSessionEvent se) {
		// TODO Auto-generated method stub
		if(sessions.get(se.getSession().getId()) != null) {
			sessions.get(se.getSession().getId()).invalidate();
			sessions.remove(se.getSession().getId());
		}
	}
}
```  

- SessionConfig 클래스 getSessionCheck 메서드 구현


  
```java
@Controller
public class LoginController {

    @Resource(name = "loginService")
    private LoginService loginSerivce;

    @RequestMapping(value = "/login")
    public String login (@RequestParam  HashMap<Object, Object> param, HttpSession session){
        HashMap<Object, Object> vo = loginSerivce.selectLogin(param);
        String userId = SessionConfig.getSessionidCheck("user_id", vo.get("user_id").toString());
        session.setMaxInactiveInterval(60*60);
        session.setAttribute("user_id", vo.get("user_id").toString());
        return "main";
    }
}
```  

- Controller 클래스에서 로그인 시 SessionConfig.getSessionidCheck() 메서드 호출
