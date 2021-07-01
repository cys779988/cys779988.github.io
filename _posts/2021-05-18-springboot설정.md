---
title:  "spring boot application.properties"
excerpt: spring boot application.properties 설정
categories:
  - spring
---

## mysql연동 설정
  
```
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/test?serverTimezone=UTC&characterEncoding=UTF-8
spring.datasource.username=root
spring.datasource.password=1234
```  

## jpa 설정
  
```
spring.jpa.database-platform=org.hibernate.dialect.MySQL5InnoDBDialect
spring.jpa.open-in-view=false
spring.jpa.show-sql=true
spring.jpa.hibernate.format_sql=true
spring.jpa.hibernate.ddl-auto=update
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE
```  
### spring.jpa.database-platform
- JPA 데이터베이스 플랫폼을 지정

### spring.jpa.open-in-view
- OSIV(Open Session In View)는 웹 요청이 완료될 때까지 동일한 EntityManager를 갖도록 해줌
- 스프링부트 기본값은 true, 성능과 확장성 면에서 안 좋기 때문에 false로 설정

### spring.jpa.show-sql
- 콘솔에 JPA 실행 쿼리를 출력

### spring.jpa.hibernate.format_sql
- 콘솔에 출력되는 JPA실행 쿼리를 가독성있게 표현

### spring.jpa.hibernate.ddl-auto 옵션
- none : 아무것도 실행하지 않음
- create-drop : SessionFactory가 시작될 때 drop 및 생성을 실행, SessionFactory가 종료될 때 drop을 실행
- create : SessionFactory가 시작될 때 데이터베이스 drop을 실행하고 생성된 DDL을 실행
- update : 변경된 스키마만 적용
- validate : 변경된 스키마가 있다면 변경점을 출력하고 애플리케이션을 종료

### logging.level.org.hibernate.type.descriptor.sql
- SQL에서 물음표로 표기된 부분(bind parameter)을 로그로 출력

### spring.jpa.hibernate.naming
- 엔티티와 테이블에 대한 네이밍 전략

### spring.jpa.hibernate.use-new-id-generator-mappings
- auto increment에 대한 설정


## httpMethod 설정
@DeleteMapping, @PutMapping 사용할 경우 다음 코드를 application.properties에 작성

  
```
spring.mvc.hiddenmethod.filter.enabled=true
```  

설정 사용 전
  
```
 <input type="hidden" name="_method" value="put"/>
```  

설정 사용 후
  
```
 <form th:action="@{'/post/edit/' + ${boardDto.id}}" th:method="put">
```  

\<input\> 태그를 생략하고 th:method에서 PUT또는 DELETE를 사용해서 보내는 \_method를 사용해서 @PutMapping과 @DeleteMapping으로 요청을 매핑할 수 있음
