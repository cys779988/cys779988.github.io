---
title:  "DataAccess"
excerpt: DataAccess
categories:
  - eGovFrame
---

### DataAccess
- JDBC 를 사용한 Data Access를 추상화하여 간편하고 쉽게 사용할 수 있는 Data Mapper framework 인 iBATIS 를 Data Access 기능의 기반 오픈 소스로 채택
- iBATIS 를 사용하면 관계형 데이터베이스에 엑세스하기 위해 필요한 일련의 자바 코드 사용을 현저히 줄일 수 있으며 간단한 XML 기술을 사용하여 SQL 문을 JavaBeans (또는 Map) 에 간편하게 매핑할 수 있음
- Data Access 서비스는 다양한 데이터베이스 솔루션 및 데이터베이스 접근 기술에 일관된 방식으로 대응하기 위한 서비스
  - 데이터를 조회하거나 입력, 수정, 삭제하는 기능을 수행하는 메커니즘을 단순화함
  - 데이터베이스 솔루션이나 접근 기술이 변경될 경우에도 데이터를 다루는 시스템 영역의 변경을 최소화할 수 있도록 데이터베이스와의 접점을 추상화함
  - 추상화된 데이터 접근 방식을 템플릿(Template)으로 제공함으로써, 개발자들의 업무 효율을 향상시킴.
  
### iBatis
- 단순성이라는 사상을 강조한 퍼시스턴스 프레임워크로, SQL 맵을 이용하여 반복적이고 복잡한 DB 작업 코드를 최소화함
- 단순성이라는 사상을 강조하여, XML을 이용하여 Stored Procedure 혹은 SQL 문과 자바 객체간의 매핑을 지원
- 소스코드 외부에 정의된 SQL문과 설정 정보를 바탕으로, 객체와 테이블 간의 매핑 기능을 제공
  - iBATIS Data Mapper API 는 XML을 사용하여 SQL문과 객체 매핑 정보를 간편하게 기술할 수 있도록 지원
  - 자바빈즈 객체와 Map 구현체, 다양한 원시 래퍼 타입(String, Integer..) 등을 PreparedStatement 의 파라미터나 ResultSet에 대한 결과 객체로 쉽게 매핑해 줌
  
<img src="https://cys779988.github.io/assets/img/egov-30.PNG">  

### iBatis 주요기능
- 추상화된 접근 방식 제공
  - JDBC 데이터 엑세스에 대한 추상화된 접근 방식으로, 간편하고 쉬운 API, 자원 연결/해제, 공통 에러 처리 등을 통합 지원함
- 자바 코드로부터 SQL문 분리
  - 소스코드로부터 SQL문을 분리하여 별도의 repository(의미있는 문법의 XML)에 유지하고 이에 대한 빠른 참조구조를 내부적으로 구현하여 관리/유지보수/튜닝의 용이성을 보장함.
- SQL문 자동 실행, 입/출력 파라미터 자동 바인딩 지원
  - 쿼리문의 입력 파라미터에 대한 바인딩과 실행결과 resultset 의 가공(맵핑) 처리시 객체(VO, Map, List) 수준의 자동화를 지원함
- Dynamic SQL 지원
  - 코드 작성, API 직접 사용없이 입력 조건에 따른 동적인 쿼리문 변경을 지원함
- 다양한 DB 처리 지원
  - 기본 질의 외에 Batch SQL, Paging, Callable Statement, BLOB/CLOB 등 다양한 DB처리를 지원함
  
### iBatis를 사용한 Persistence Layer 개발 순서
<img src="https://cys779988.github.io/assets/img/egov-31.PNG">  

### SQL Mapping XML 파일 작성
실행할 SQL문과 Parameter Object와 Result Object, Dynamic SQL 등을 설정
<img src="https://cys779988.github.io/assets/img/egov-32.PNG">  
<img src="https://cys779988.github.io/assets/img/egov-33.PNG">  

### iBatis Configuration XML 파일
- iBatis 공통 설정 파일로 SqlMapClient 설정관련 상세 내역을 제어할 수 있는 메인 설정
- 주로 transaction 관리 관련 설정 및 다양한 옵션 설정, Sql Mapping 파일들에 대한 path 설정 등을 포함
<img src="https://cys779988.github.io/assets/img/egov-34.PNG">  
<img src="https://cys779988.github.io/assets/img/egov-35.PNG">  
  
요소 | 설명
---- | ----
properties | 표준 java properties (key=value 형태)파일에 대한 연결을 지원하며 설정 파일내에서 ${key} 와 같은 properties 형태로 외부화 놓은 실제의 값(여기서는 DB 접속 관련 driver, url, id/pw)을 참조할 수 있다. resource 속성으로 classpath 지정 가능, url 속성으로 유효한 URL 상에 있는 자원을 지정 가능
settings | 이 설정 파일을 통해 생성된 SqlMapClient instance 에 대하여 다양한 옵션 설정을 통해 최적화할 수 있도록 지원한다. 모든 속성 선택사항(optional) 이다.
typeHandler | javaType과 jdbcType 일치를 위해 TypeHandler 구현체를 등록하여 사용할 수 있다.
sqlMap | 매핑할 SQL구문이 정의된 파일 지정한다.
  
- \<settings\>에서 사용 가능한 속성들

  
속성 | 설명 | 예시 및 디폴트값
---- | ---- | ----
maxRequests | 같은 시간대에 SQL 문을 실행할 수 있는 thread 의 최대 갯수 지정 | maxRequests=“256”, 512
maxSessions | 주어진 시간에 활성화될 수 있는 session(또는 client) 수 지정 | maxSessions=“64”, 128
maxTransactions | 같은 시간대에 SqlMapClient.startTransaction() 에 들어갈 수 있는 최대 갯수 지정 | maxTransactions=“16”, 32
cacheModelsEnabled | SqlMapClient 의 모든 cacheModel 에 대한 사용 여부를 global하게 지정 | cacheModelsEnabled=“true”, true (enabled)
lazyLoadingEnabled | SqlMapClient 의 모든 lazy loading 에 대한 사용 여부를 global하게 지정 | lazyLoadingEnabled=“true”, true (enabled)
enhancementEnabled | runtime bytecode enhancement 기술 사용 여부 지정 | enhancementEnabled=“true”, false (disabled)
useStatementNamespaces | Statement 호출 시 namespace값 사용 여부 | useStatementNamespaces=“false”, false (disabled)
defaultStatementTimeout | 모든 JDBC 쿼리에 대한 timeout 시간(초) 지정, 각 statement 의 설정으로 override 가능함. 모든 driver가 이 설정을 지원하는 것은 아님에 유의할 것 | 지정하지 않는 경우 timeout 없음(cf. 각 statement 설정에 따라)
classInfoCacheEnabled | introspected(java 의 reflection API에 의해 내부 참조된) class의 캐쉬를 유지할지에 대한 설정 | classInfoCacheEnabled=“true”, true (enabled)
statementCachingEnabled | prepared statement 의 local cache 를 유지할지에 대한 설정 | statementCachingEnabled=“true”, true (enabled)
  

### (스프링연동 설정)SqlMapClientFactoryBean 정의
- Spring과 iBatis 연동을 위한 설정으로, iBatis 관련 메서드 실행을 위해 SqlMapClient 객체가 필요
- 스프링에서 SqlMapClient 객체를 생성하고 관리할 수 있도록, SqlMapClientFactoryBean을 정의
  - Id와 class는 고정값
  - dataSource : 스프링에서 설정한 Datasource Bean id를 설정하여 iBatis가 DataSource를 사용하게 한다.
  - configLocation : iBatis Configuration XML 파일이 위치하는 곳을 설정한다.
  - mappingLocations : SQL Mapping XML 파일을 일괄 지정할 수 있다. 단, Configuration 파일에 중복 선언할 수 없다.
  - 실행환경 3.5 부터는 Spring 4 변경사항에 의해 org.springframework.orm.ibatis.SqlMapClientFactoryBean 클래스가
egovframework.rte.psl.orm.ibatis.SqlMapClientFactoryBean 로 변경됨.

<img src="https://cys779988.github.io/assets/img/egov-36.PNG">  

### iBatis 활용한 자바클래스 작성
EgovAbstractDAO 클래스를 상속받아 DAO 클래스를 작성
<img src="https://cys779988.github.io/assets/img/egov-37.PNG">  

### 세부사항 설명
- iBATIS Configuration
  - iBATIS 의 메인 설정 파일인 SQL Map XML Configuration 파일(이하 sql-map-config.xml 설정 파일) 작성과 상세한 옵션 설정
- Data Type
  - 데이터베이스를 이용하여 데이터를 저장하고 조회할 때 Java 어플리케이션에서의 Type 과 DBMS 에서 지원하는 관련 매핑 jdbc Type 의 정확한 사용이 필요
- parameterMap
  - 해당 요소로 SQL 문 외부에 정의한 입력 객체의 속성에 대한 name 및 javaType, jdbcType 을 비롯한 옵션을 설정할 수 있는 매핑 요소
- Inline parameters
  - prepared statement 에 대한 바인드 변수 매핑 처리를 위한 parameterMap 요소(SQL 문 외부에 정의한 입력 객체 property
name 및 javaType, jdbcType 을 비롯한 옵션을 설정매핑 요소) 와 동일한 기능을 처리하는 간편한 방법
- resultMap
  - resultMap 은 SQL 문 외부에 정의한 매핑 요소로, result set 으로부터 어떻게 데이터를 뽑아낼지, 어떤 칼럼을 어떤
property로 매핑할지에 대한 상세한 제어를 가능케 해줌
- Dynamic SQL
  - SQL 문의 동적인 변경에 대한 상대적으로 유연한 방법을 제공하는 iBATIS 의 Dynamic 요소

### Dynamic SQL
- 일반적으로 JDBC API 를 사용한 코딩에서 한번 정의한 쿼리문을 최대한 재사용하고자 하나 단순 파라메터
변수의 값만 변경하는 것으로 해결하기 어렵고 다양한 조건에 따라 조금씩 다른 쿼리의 실행이 필요한 경우
많은 if~else 조건 분기의 연결이 필요한 문제가 있음.
  
<img src="https://cys779988.github.io/assets/img/egov-38.PNG">  

- Unary 비교 연산
<img src="https://cys779988.github.io/assets/img/egov-39.PNG">  

- Unary 비교 연산 태그
  
태그 | 설명
---- | ----
isEmpty | Collection, String(또는 String.valueOf()) 대상 속성이 null 이거나 empty(”” 또는 size() < 1) 인 경우 true
isNotEmpty | Collection, String(또는 String.valueOf()) 대상 속성이 not null 이고 not empty(”” 또는 size() < 1) 인 경우 true
isNull | 대상 속성이 null 인 경우 true
isNotNull | 대상 속성이 not null 인 경우 true
isPropertyAvailable | 파라메터 객체에 대상 속성이 존재하는 경우 true
isNotPropertyAvailable | 파라메터 객체에 대상 속성이 존재하지 않는 경우 true
  
- Unary 비교 연산 태그 속성
  
속성 | 설명
---- | ----
prepend | 동적 구문 앞에 추가되는 override 가능한 SQL 영역
property | 필수. 파라메터 객체의 어떤 property 에 대한 체크인지 지정
removeFirstPrepend | 첫번째로 내포될 내용을 생성하는 태그의 prepend 를 제거할지 여부(true/false)
open | 전체 결과 구문에 대한 시작 문자열
close | 전체 결과 구문에 대한 닫는 문자열
  
- Binary 비교 연산
<img src="https://cys779988.github.io/assets/img/egov-40.PNG">  

- Binary 비교 연산 태그
  
태그 | 설명
---- | ----
isEqual | 대상 속성이 compareValue 값 또는 compareProperty 로 명시한 대상 속성 값과 같은 경우 true 
isNotEqual | 대상 속성이 compareValue 값 또는 compareProperty 로 명시한 대상 속성 값과 다른 경우 true 
isGreaterEqual | 대상 속성이 compareValue 값 또는 compareProperty 로 명시한 대상 속성 값보다 크거나 같은 경우 true 
isGreaterThan | 대상 속성이 compareValue 값 또는 compareProperty 로 명시한 대상 속성 값보다 큰 경우 true 
isLessEqual | 대상 속성이 compareValue 값 또는 compareProperty 로 명시한 대상 속성 값보다 작거나 같은 경우 true 
isLessThan | 대상 속성이 compareValue 값 또는 compareProperty 로 명시한 대상 속성 값보다 작은 경우 true 
  
- Binary 비교 연산 태그 속성
  
속성 | 설명
---- | ----
prepend | 동적 구문 앞에 추가되는 override 가능한 SQL 영역
property | 필수. 파라메터 객체의 어떤 property에 대한 비교인지 지정
compareProperty | 파라메터 객체의 다른 property와 대상 property 값을 비교하고자 할 경우 지정. (compareValue 가 없는 경우 필수) 
compareValue | 대상 property 와 비교될 값을 지정. (compareProperty 가 없는 경우 필수)
removeFirstPrepend | 첫번째로 내포될 내용을 생성하는 태그의 prepend 를 제거할지 여부(true/false) 
open | 전체 결과 구문에 대한 시작 문자열
close | 전체 결과 구문에 대한 닫는 문자열
  

- ParameterPresent 비교
<img src="https://cys779988.github.io/assets/img/egov-41.PNG">  

- ParameterPresent 비교 태그
  
태그 | 설명
---- | ----
isParameterPresent | 파라메터 객체가 전달된(not null) 경우 true
isNotParameterPresent | 파라메터 객체가 전달되지 않은(null) 경우 true
  
- ParameterPresent 비교 태그 속성
  
속성 | 설명
---- | ----
prepend | 동적 구문 앞에 추가되는 override 가능한 SQL 영역
property | 필수. 파라메터 객체의 어떤 property 에 대한 비교인지 지정
removeFirstPrepend | 첫번째로 내포될 내용을 생성하는 태그의 prepend 를 제거할지 여부(true/false/iterate)
open | 전체 결과 구문에 대한 시작 문자열
close | 전체 결과 구문에 대한 닫는 문자열
  
- Sample iterate 연산
<img src="https://cys779988.github.io/assets/img/egov-42.PNG">  

- iterate 연산 태그
  
태그 | 설명
---- | ----
iterate collection | 형태의 대상 객체에 대하여 포함하고 있는 각 개별 요소만큼 반복 루프를 돌며 해당 내용을 수행함
  
- iterate 연산 태그 속성
  
속성 | 설명
---- | ----
prepend | 동적 구문 앞에 추가되는 override 가능한 SQL 영역
property | 필수. 파라메터 객체의 어떤 property 에 대한 비교인지 지정
removeFirstPrepend | 첫번째로 내포될 내용을 생성하는 태그의 prepend 를 제거할지 여부(true/false/iterate)
open | 전체 결과 구문에 대한 시작 문자열
close | 전체 결과 구문에 대한 닫는 문자열
conjunction | 각 iteration 사이에 적용될 문자열. AND, OR 연산자나 ',' 등의 구분자 필요 시 유용함
  

### MyBatis 데이터 매퍼 서비스
- 개발자가 작성한 SQL문 혹은 저장프로시저 결과값을 자바 오브젝트에 자동 매핑하는 서비스
- 수동적인 JDBC 방식의 데이터 처리 작업 코드와는 달리 쿼리결과와 오브젝트 간 자동 매핑을 지원
- SQL문과 저장프로시저는 XML 혹은 어노테이션 방식으로 작성 가능

<img src="https://cys779988.github.io/assets/img/egov-43.PNG">  

### 주요 변경 사항
- iBatis의 SqlMapClient -> SqlSession 변경
  - SqlSession 인터페이스
    - MyBatis를 사용하기 위한 기본적인 인터페이스로, SQL문 처리를 위한 메서드를 제공
    - 구문 실행 메서드, 트랜잭션 제어 메서드 등 포함 selectList(), selectOne(), insert(), update(), delete(), commit(), rollback(), …
    - SqlSessionFactory 클래스를 통해 MyBatis Configuration 정보에 해당 SqlSession 인스턴스를 생성
- 어노테이션 방식 설정 도입
  - MyBatis는 본래 XML 기반의 프레임워크였으나, Mybatis 3.x 부터 어노테이션 방식의 설정을 지원
  - Mapper XML File 내 SQL문 및 매핑 정보를, 자바 코드 내에서 어노테이션으로 그대로 적용 가능
- iBatis의 RowHandler  ResultHandler 변경
  - ResultHandler 인터페이스
    - Result Object에 담겨 리턴된 쿼리 결과를 핸들링할 수 있도록 메서드 제공
    - 사용 예시) 대량의 데이터 처리 시, 처리 결과를 File로 출력하고자 할 때 혹은 Result Object의 형태를 Map 형태로 가져올 때

### Migrating from iBatis
<img src="https://cys779988.github.io/assets/img/egov-44.PNG">  
<img src="https://cys779988.github.io/assets/img/egov-45.PNG">  

### MyBatis 활용한 Persistence Layer 개발
<img src="https://cys779988.github.io/assets/img/egov-46.PNG">  

### SQL Mapper XMl 파일 작성
<img src="https://cys779988.github.io/assets/img/egov-47.PNG">  

### SQL Mapper XMl 파일 작성 - Dynamic SQL
- If
<img src="https://cys779988.github.io/assets/img/egov-48.PNG">  

- choose(when, otherwise) : 자바의 switch 구문과 유사한 개념
<img src="https://cys779988.github.io/assets/img/egov-49.PNG">  

- trim(where, set) : AND, OR, ','와 같이 반복되는 문자를 자동적으로 trim(제거)
<img src="https://cys779988.github.io/assets/img/egov-50.PNG">  

- foreach : Map, List, Array에 담아 넘긴 값을 꺼낼 때 사용하는 요소
<img src="https://cys779988.github.io/assets/img/egov-51.PNG">  

### MyBatis Configuration XML 파일 작성
- MyBatis 공통 설정 파일로, SqlSession 설정관련 상세 내역을 제어할 수 있는 메인 설정
<img src="https://cys779988.github.io/assets/img/egov-52.PNG">  

  
요소 | 설명
---- | ----
properties | 설정 파일 내에서 ${key} 와 같은 형태로 외부 properties 파일을 참조할 수 있다.
settings | 런타임시 MyBatis의 행위를 조정하기 위한 옵션 설정을 통해 최적화할 수 있도록 지원한다
typeAliases | 타입 별칭을 통해 자바타입에 대한 좀더 짧은 이름을 사용할 수 있다. 오직 XML 설정에서만 사용되며, 타이핑을 줄이기 위해 사용된다.
typeHandlers | javaType과 jdbcType 일치를 위해 TypeHandler 구현체를 등록하여 사용할 수 있다.
environments | 환경에 따라 MyBatis 설정을 달리 적용할 수 있도록 지원한다.
Mappers | 매핑할 SQL 구문이 정의된 파일을 지정한다.
  

### (스프링연동 설정)SqlSessionFactoryBean 정의
- Spring와 MyBatis 연동을 위한 설정으로, MyBatis 관련 메서드 실행을 위해 SqlSession 객체가 필요
- 스프링에서 SqlSession 객체를 생성하고 관리할 수 있도록, SqlSessionFactoryBean을 정의
  - Id와 class는 고정값
  - dataSource : 스프링에서 설정한 Datasource Bean id를 설정하여 MyBatis가 DataSource를 사용하게 한다.
  - configLocation : MyBatis Configuration XML 파일이 위치하는 곳을 설정한다.
  - mapperLocations : SQL Mapper XML 파일을 일괄 지정할 수 있다. 단, Configuration 파일에 중복 선언할 수 없다.

<img src="https://cys779988.github.io/assets/img/egov-53.PNG">  

### MyBatis 활용한 자바클래스 작성(DAO 클래스 대신 Interface 작성 (Mapper Interface 방식))

- EgovAbstractMapper 클래스를 상속받아 DAO 클래스를 작성
<img src="https://cys779988.github.io/assets/img/egov-54.PNG">  

- 기존 DAO 클래스의 MyBatis 메서드 호출 코드를 최소화시킨 방법으로, 각 Statement id와 메서드명을 동일하게 작성하면 MyBatis가 자동으로 SQL문을 호출한다.
- 실제 내부적으로 MyBatis는 풀네임을 포함한 메서드명을 Statement id로 사용한다
- namespace : 각 SQL Mapper XML을 구분
<img src="https://cys779988.github.io/assets/img/egov-55.PNG">  
<img src="https://cys779988.github.io/assets/img/egov-56.PNG">  

- @Mapper를 사용하여 Mapper Interface가 동작하도록 하려면, MapperConfigurer 클래스를 빈으로 등록한다.
- MapperConfigurer는 @Mapper를 자동 스캔하고, MyBatis 설정의 편리함을 제공한다.
- basePackage : 스캔 대상에 포함시킬 Mapper Interface가 속한 패키지를 지정
<img src="https://cys779988.github.io/assets/img/egov-57.PNG">  

- 어노테이션을 이용한 SQL문 작성
- 인터페이스 메소드 위에 @Statement(Select, Insert, Update, Delete …)를 선언하여 쿼리를 작성한다.
- SQL Mapper XML을 작성할 필요가 없으나, Dynamic 쿼리를 사용하지 못하고 쿼리의 유연성이 떨어진다.
<img src="https://cys779988.github.io/assets/img/egov-58.PNG">  
