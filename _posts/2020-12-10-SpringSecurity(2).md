---
title:  "spring Sercurity(2)"
excerpt: spring Sercurity 공부
categories:
  - spring
---

## 스프링 시큐리티 커스터마이징
1. 원하는 스타일의 로그인 화면으로 교체
2. 로그인 에러 메세지를 원하는대로 변경
3. 원하는 스타일의 접근 권한 에러 화면 변경
4. DB를 이용한 사용자 인증 절차
5. 패스워드 암호화
6. DB를 통해 접근 경로 권한 관리
7. 권한 체크 시 메모리에서 체크 

## 3가지 인터페이스 상속
  
```java
org.springframework.security.core.userdetails.UserDetails

org.springframework.security.core.userdetails.UserDetailsService

org.springframework.security.core.authentication.AuthenticationProvider
```  

- org.springframework.security.core.authentication.AuthenticationProvider
인증 절차가 시작되면 AuthenticationProvider 인터페이스가 실행되는데, 여기에서 DB에 있는 이용자의 정보와 화면에서 입력한 로그인 정보를 비교하게 된다.  
AuthenticationProvider 인터페이스에서는 authenticatie() 메소드를 오버라이드 하게 되는데, 이 메소드의 파라미터인 Authentication 으로 화면에서 입력한 로그인 정보를 가져올 수 있다.

- org.springframework.security.core.userdetails.UserDetailsService
AuthenticationProvider 인터페이스에서 DB에 있는 이용자의 정보를 가져오려면, UserDetailsService 인터페이스를 사용
UserDetailsService 인터페이스는 화면에서 입력한 이용자의 이름(username)을 가지고 loadUserByUsername() 메소드를 호출하여 DB에 있는 이용자의 정보를 UserDetails 형으로 가져온다.  
만약 이용자가 존재하지 않으면 예외를 던진다. 이렇게 DB에서 가져온 이용자의 정보와 화면에서 입력한 로그인 정보를 비교하게 되고, 일치하면 Authentication 참조를 리턴하고, 일치 하지 않으면 예외를 던진다.


## 경로에 따른 접근 권한
### 권한은 비회원(GUEST), 준회원(USER), 정회원(MEMBER), 관리자(ADMIN) 4가지  
#### 비회원은 그 누구든 접근  
#### 준회원은 로그인을 한 이용자  
#### 정회원은 로그인 한 이용자 중에 정회원 권한을 가진 이용자  
#### 관리자는 로그인 한 이용자 중에 관리자 권한을 가진 이용자
  
```
<http auto-config="true" use-expressions="true">
    <intercept-url pattern="/member/**" access="hasAnyRole('ROLE_MEMBER','ROLE_ADMIN')"/>
    <intercept-url pattern="/user/**" access="hasAnyRole('ROLE_USER','ROLE_MEMBER','ROLE_ADMIN')"/>
    <intercept-url pattern="/admin/**" access="hasRole('ROLE_ADMIN')"/>
    <intercept-url pattern="/**" access="permitAll"/>
</http>
```  
<intercept-url> 태그를 사용할 때 반드시 주의해야할 점이 있다. 경로 설정을 하게 되면 태그를 하나가 아닌 여러개를 설정하게 된다. 여러개가 설정할 경우, 설정된 순서대로 위에서부터 매칭되어진다.  
따라서 태그의 설정 순서가 중요하다. 태그의 순서로 인해서 꼬여버리거나 무한 루프가 돌 수 있기 때문이다. 가장 특수한 경우를 위에 놓고 아래쪽으로 갈 수록 일반적인 경우를 둬야 한다.  
즉, 구체적인 패턴을 먼저 설정해야 한다. /admin/** 패턴과 /** 패턴를 보면, /admin/** 패턴은 admin 디렉토리 밑에 있는 모든 디렉토리와 파일을 의미하고, /** 패턴은 모든 디렉토리와 파일을 의미한다.  
즉, /admin/** 은 /**보다 더 구체적인 패턴이라는 뜻이다. 그럼 /admin/** 패턴을 /** 보다 위에 설정해주어야 한다. 만약 /** 패턴을 먼저 설정해버리면 모든 url이 /** 패턴을 만족하게 되고, admin 하위 디렉토리를 가르켜도 /** 패턴을 먼저 매칭시켰기 때문에 admin이 아닌 권한들도 admin 디렉토리에 접근이 가능해져 버린다.  
따라서 구체적인 패턴이 먼저 설정되어야 한다.




