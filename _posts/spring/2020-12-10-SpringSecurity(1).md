---
title:  "spring Sercurity"
excerpt: spring Sercurity 공부
categories:
  - spring
---

## 스프링 시큐리티(Spring Security)
- 스프링 시큐리티는 스프링 기반의 애플리케이션의 보안을 담당하는 프레임워크
- 보안과 관련된 많은 옵션들을 지원
- 스프링과 완전 분리되어 Filter 기반으로 동작
- 세션 : 쿠키 방식으로 인증(요청을 받아 DB에 검증된 회원일 경우 JSESSIONID 부여 후 요청에서 JSESSIONID 검증 후 유효하면 인증을 줌)

## 스프링 시큐리티 구성 영역
- Authentication(인증) : : 'A'라고 주장하는 주체(user, subject, principal)가 'A'가 맞는지 확인하는 것
  - 코드에서 Authentication : 인증 과정에 사용되는 핵심 객체
  - ID/PASSWORD, JWT, OAuth 등 여러 방식으로 인증에 필요한 값이 전달되는데 이것을 하나의 인터페이스로 받아 수행하도록 추상화 하는 역할의 인터페이스
- Authorization(권한) : 특정 자원에 대한 권한이 있는지 확인하는 것
  - 프로세스상 신분 "인증"을 거치고 신분 인증이 되었으면 권한이 있는지 확인 후, 서버 자원에 대해서 접근할 수 있게 되는 순서
  - ID/PASSWORD, 공인인증서, 지문 등으로 로그인을 하는 것은 '인증'에 해당한다.
- Credential(증명서) : 인증 과정 중, 주체가 본인을 인증하기 위해 서버에 제공하는 것 (ID, PASSWORD)

## 스프링 시큐리티 기본아키텍쳐
1. REQUEST 받음
2. Authentication Filter에 걸림
3. 만약 처리 가능한 필터가 없으면 인증되지 않은 요청으로 취급. 서비스에 따라 인증되지 않은 요청에 대한 로직 구현
4. 처리가능한 필터를 찾으면 AuthenticateManager 호출
5. AuthenticationManager 안에서 역할에 따라 위임된 각각의 객체를 이용해 인증처리  
즉, AuthenticationManager의 구현체인 ProviderManager에서 AuthenticationProvider를 순회하며 인증처리 위임

<img src="https://cys779988.github.io/assets/img/springsec-1.PNG">

Spring Sercurity는 필터를 수행하고 다음 필터의 일을 수행하는 체인 형식으로 구성

## Spring Framework 시큐리티 사용자 인증 적용

### 의존성 추가
  
```
<!-- Spring Security -->
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-core</artifactId>
    <version>4.2.0.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-config</artifactId>
    <version>4.2.0.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-taglibs</artifactId>
    <version>4.2.0.RELEASE</version>
</dependency>

<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-acl</artifactId>
    <version>4.2.0.RELEASE</version>
</dependency>
```  

### web.xml 설정
- Spring Security가 모든 요청을 가로채 보안이 적용되게 하는 서블릿 필터
- 보안 처리와 관련된 일을 하는 것은 아니지만, 보안 적용을 위해 spring security에게 권한 부여 등을 체크하기 위해 넘겨주는 역할
- springSecurityFilterChain는 chain형식으로 이루어져 각각의 filter들이 순차적으로 일을 수행(SpringSecurityFilterChain이 Bean에 들어가 있어야함.)
  
```
<!-- Spring Security Filter -->
<filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>
<filter-mapping>
    <filter-name>springSecurityFilterChain</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```  

### context-security.xml 설정
  
```
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:security="http://www.springframework.org/schema/security"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/security
           http://www.springframework.org/schema/security/spring-security.xsd">

	<security:http use-expressions="true">
		<security:intercept-url
			pattern="/**" access="isAuthenticated()" />
		<security:form-login login-page="/member/loginForm.do"
			login-processing-url="/member/login.do" username-parameter="empNo"
			password-parameter="pwd"
			authentication-failure-handler-ref="customAuthenticationFailHandler" />

		<security:logout logout-url="/member/logout.do"
			logout-success-url="/" />
	</security:http>
</beans>
```  

- isAuthenticated() : 인증,허가된 사용자만 접근
- isAnonymous() : 익명 사용자만 접근
- permitAll : 인증을 했건 안했건 모두 접근이 가능하도록 허락
- login-page="/member/loginForm.do" : 사용자가 직접 만든 로그인페이지 사용 (loginForm.do 경로에 대해서만 스프링이 로그인 처리를 해주겠다는 의미)
- login-processing-url="/member/login.do" : form태그에서 전송버튼을 눌렀을때 action을 /member/login.do 경로로 하면 이제부터 스프링 시큐리티가 인증 절차를 진행하겠다는 뜻
- username-parameter="empNo" : form 의 empNo 값을 파라미터로 사용
- password-parameter="pwd" : form 의 pwd 값을 파라미터로 사용
- authentication-failure-handler-ref="customAuthenticationFailHandler" : 인증 실패했을 때 처리
- \<security:logout logout-url="/member/logout.do" logout-success-url="/"/\> : /member/logout.do 라는 요청 경로가 들어왔을 때 로그아웃 처리, 성공하면 / 경로로 이동
  

## Spring Boot 시큐리티 사용자 인증 적용

### 의존성 추가

build.gradle
  
```
implementation 'org.springframework.boot:spring-boot-starter-security'
implementation 'org.thymeleaf.extras:thymeleaf-extras-springsecurity5'
```  

### Spring Security 설정
WebSecurityConfigurerAdapter 클래스를 상속받은 클래스에서 메서드를 오버라이딩하여 조정
  
```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.builders.WebSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.util.matcher.AntPathRequestMatcher;

import com.spring.member.MemberService;

import lombok.AllArgsConstructor;

@Configuration
@EnableWebSecurity
@AllArgsConstructor
public class SecurityConfig extends WebSecurityConfigurerAdapter{
	private MemberService memberService;
	
	@Bean
	public PasswordEncoder passwordEncoder() {
		return new BCryptPasswordEncoder();
	}
	
	@Override
	public void configure(WebSecurity web) throws Exception {
		// TODO Auto-generated method stub
		web.ignoring().antMatchers("/css/**", "/js/**", "/img/**", "/lib/**");
	}
	
	@Override
	protected void configure(HttpSecurity http) throws Exception {
		// TODO Auto-generated method stub
		http.authorizeRequests()
				// 페이지권한설정
				.antMatchers("/admin/**").hasRole("ADMIN")
				.antMatchers("/user/myinfo").hasRole("MEMBER")
				.antMatchers("/**").permitAll()
			.and()	// 로그인설정
				.formLogin()
				.loginPage("/user/login")
				.defaultSuccessUrl("/user/login/result")
				.permitAll()
			.and()	// 로그아웃설정
				.logout()
				.logoutRequestMatcher(new AntPathRequestMatcher("/user/logout"))
				.logoutSuccessUrl("/user/logout/result")
				.invalidateHttpSession(true)
			.and()	// 예외처리
				.exceptionHandling().accessDeniedPage("/user/denied");
	}
	
	@Override
	protected void configure(AuthenticationManagerBuilder auth) throws Exception {
		// TODO Auto-generated method stub
		auth.userDetailsService(memberService).passwordEncoder(passwordEncoder());
	}
}

```  

#### @EnableWebSecurity
- @Configuration 클래스에 @EnableWebSecurity 어노테이션 추가하여 Spring Security 설정 클래스로 정의
- 설정은 WebSebSecurityConfigurerAdapter 클래스를 상속받아 메서드를 구현하는 것이 일반적인 방법

#### WebSecurityConfigurerAdapter 클래스
- WebSecurityConfigurer 인스턴스를 편리하게 생성하기 위한 클래스

#### passwordEncoder()
- BCryptPasswordEncoder는 Spring Security에서 제공하는 비밀번호 암호화 객체
- Service에서 비밀번호를 암호화할 수 있도록 Bean으로 등록

#### configure(WebSecurity web)
- WebSecurity는 FilterChainProxy를 생성하는 필터
-   ```web.ignoring().antMatchers("/css/**", "/js/**", "/img/**", "/lib/**");```  
- 해당 경로의 파일들은 Spring Security가 무시할 수 있도록 설정합니다. 파일 기준은 resources/static 디렉터리

#### configure(HttpSecurity http)
- HttpSecurity를 통해 HTTP 요청에 대한 웹 기반 보안을 구성

#### authorizeRequests()
- HttpServletRequest에 따라 접근을 제한함
- antMatchers() 메서드로 특정 경로를 지정하며, permitAll(), hasRole() 메서드로 역할에 따른 접근 설정
  ```.antMatchers("/admin/**").hasRole("ADMIN")```   
- /admin 으로 시작하는 경로는 ADMIN 롤을 가진 사용자만 접근 가능
  ```.antMatchers("/user/myinfo").hasRole("MEMBER")```  
- /user/myinfo 경로는 MEMBER 롤을 가진 사용자만 접근 가능
  ```.antMatchers("/**").permitAll()```  
- 모든 경로에 대해서 권한없이 접근 가능
  ```.anyRequest().authenticated()```  
- 모든 요청에 대해 인증된 사용자만 접근 가능

#### formlogin()
- form 기반으로 인증, 로그인 정보는 기본적으로 HttpSession 이용함
- /login 경로로 접근하면 Spring Security에서 제공하는 로그인 form 사용
- .loginPage("/user/login")
  - 기본 제공되는 form 말고, 커스텀 로그인 폼을 사용하고 싶으면 loginPage() 메서드를 사용
  - 이 때 커스텀 로그인 form의 action 경로와 loginPage()의 파라미터 경로가 일치해야 인증을 처리 ( login.html에서 확인 )
- .defaultSuccessUrl("/user/login/result")
  - 로그인이 성공했을 때 이동되는 페이지, 컨트롤러에서 URL 매핑이 되어있어야함
- .usernameParameter("파라미터명")
  - 로그인 form에서 아이디는 name=username인 input을 기본으로 인식하는데, usernameParameter() 메서드를 통해 파라미터명을 변경

#### logout()
- 로그아웃을 지원하는 메서드이며, WebSecurityConfigurerAdapter를 사용할 때 자동으로 적용
- 기본적으로 "/logout"에 접근하면 HTTP 세션을 제거
- .logoutRequestMatcher(new AntPathRequestMatcher("/user/logout"))
  - 로그아웃의 기본 URL(/logout)이 아닌 다른 URL로 재정의
- .invalidateHttpSession(true)
  - HTTP 세션을 초기화하는 작업
- deleteCookies("KEY명")
  - 로그아웃 시, 특정 쿠키를 제거하고 싶을 때 사용하는 메서드

#### exceptionHandling().accessDeniedPage("/user/denied")
  - 예외가 발생했을 때 exceptionHandling() 메서드로 핸들링할 수 있음 ex) 접근권한이 없을 때 로그인페이지 이동
  
#### configure(AuthenticationManagerBuilder auth)
- Spring Security에서 모든 인증은 AuthenticationManager를 통해 이루어지며 AuthenticationManager를 생성하기 위해서는 AuthenticationManagerBuilder를 사용
- Service 클래스에서는 UserDetailsService 인터페이스를 implements하여, loadUserByUsername() 메서드를 구현
- 비밀번호 암호화를 위해 passwordEncoder를 사용


### UserDetailsService 구현한 Service 클래스

  
```java
package com.spring.security;

import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import lombok.AllArgsConstructor;

@Service
@AllArgsConstructor
public class MemberService implements UserDetailsService{
	private MemberRepository memberRepository;
	
	@Transactional
	public Long joinUser(MemberDto memberDto) {
		// TODO Auto-generated method stub
		BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
		memberDto.setPassword(passwordEncoder.encode(memberDto.getPassword()));
		
		return memberRepository.save(memberDto.toEntity()).getId();
	}

	@Override
	public UserDetails loadUserByUsername(String userEmail) throws UsernameNotFoundException {
		// TODO Auto-generated method stub
		Optional<MemberEntity> userEntityWrapper = memberRepository.findByEmail(userEmail);
		MemberEntity userEntity = userEntityWrapper.get();
		
		List<GrantedAuthority> authorities = new ArrayList<>();
		
		if(("admin@example.com").equals(userEmail)) {
			authorities.add(new SimpleGrantedAuthority(Role.ADMIN.getValue()));
		}else {
			authorities.add(new SimpleGrantedAuthority(Role.MEMBER.getValue()));
		}

		return new User(userEntity.getEmail(), userEntity.getPassword(), authorities);
	}
}

```  

#### loadUserByUsername()
- 상세정보를 조회하는 메서드이며 사용자의 계정정보와 권한을 갖는 UserDetails 인터페이스를 반환해야함
- 매개변수는 로그인 시 입력한 아이디, 엔티티의 PK를 뜻하는게 아니고 유저를 식별할 수 있는 어떤 값을 의미. Spring Security에서는 username을 사용
- authorities.add(new SimpleGrantedAuthority())
	- 롤을 부여하는 코드. 회원가입할 때 Role을 정할 수 있도록 Role Entity를 만들어서 매핑
	- 위 코드에서는 복잡성을 줄이기 위해 아이디가 "admin@example.com" 일 경우 ADMIN 롤 부여
- new User()
	- return은 SpringSecurity에서 제공하는 UserDetails를 구현한 User(org.springframework.security.core.userdetails.User)를 반환
	- 생성자의 각 매개변수는 아이디, 비밀번호, 권한리스트


### index.html

  
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.w3.org/1999/xhtml" xmlns:sec="http://www.w3.org/1999/xhtml">
<head>
    <meta charset="UTF-8">
    <title>메인</title>
</head>
<body>
	<h1>메인 페이지</h1>
	<hr>
	<a sec:authorize="isAnonymous()" th:href="@{/user/login}">로그인</a>
	<a sec:authorize="isAuthenticated()" th:href="@{/user/logout}">로그아웃</a>
	<a sec:authorize="isAnonymous()" th:href="@{/user/signup}">회원가입</a>
	<a sec:authorize="hasRole('ROLE_MEMBER')" th:href="@{/user/info}">내정보</a>
	<a sec:authorize="hasRole('ROLE_ADMIN)" th:href="@{/admin}">관리자</a>
</body>
</html>
```  

-   ```sec:authorize```   를 사용하여 사용자의 Role에 따라 보이는 메뉴를 다르게 설정
	- isAnonymous()
		- 익명의 사용자일 경우 로그인, 회원가입 버튼을 노출시킴
	- isAuthenticated()
		- 인증된 사용자일 경우 로그아웃 버튼을 노출시킴
	- hasRole()
		- 특정 롤을 가진 사용자에 대해 메뉴를 노출시킴

### login.html

  
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.w3.org/1999/xhtml">
<head>
    <meta charset="UTF-8">
    <title>로그인 페이지</title>
</head>
<body>
    <h1>로그인</h1>
    <hr>

    <form action="/user/login" method="post">
        <input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}" />

        <input type="text" name="username" placeholder="이메일 입력해주세요">
        <input type="password" name="password" placeholder="비밀번호">
        <button type="submit">로그인</button>
    </form>
</body>
</html>
```  

-   ```<input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}" />```  
	- form에 히든타입으로 csrf 토큰 값을 넘겨줌
	- Spring Security가 적용되면 POST방식으로 보내는 모든 데이터는 csrf 토큰 값이 필요함
