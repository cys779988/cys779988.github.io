---
title:  "spring mybatis"
excerpt: spring-mybatis 연동
categories:
  - mybatis
---

### Mybatis 설정파일 구성

- mybatis 관련 .jar 파일 lib폴더에 생성
- WEB-INF/config 밑에 action-mybatis.xml / action-service.xml / jdbc.properties 설정
- WebContent 밑에 action-servlet.xml / web.xml 설정

<img src="https://cys779988.github.io/assets/img/mybatis-1.png">
<img src="https://cys779988.github.io/assets/img/mybatis-2.png">
<img src="https://cys779988.github.io/assets/img/mybatis-3.png">
<img src="https://cys779988.github.io/assets/img/mybatis-4.png">
<img src="https://cys779988.github.io/assets/img/mybatis-5.png">
<img src="https://cys779988.github.io/assets/img/mybatis-6.png">
<img src="https://cys779988.github.io/assets/img/mybatis-7.png">
<img src="https://cys779988.github.io/assets/img/mybatis-8.png">
  
MultiActionController 패키지의 bind() 메소드 사용해서 바인딩  

<img src="https://cys779988.github.io/assets/img/mybatis-9.png">
<img src="https://cys779988.github.io/assets/img/mybatis-10.png">
  
action-mybaits.xml에서 설정한 sqlSession 빈
스프링에서 Mybatis 사용시 자동 Commit이므로 Commit 할 필요없음



