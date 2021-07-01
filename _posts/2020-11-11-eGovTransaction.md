---
title:  "Transaction"
excerpt: Transaction 기능
categories:
  - eGovFrame
---

### eGovFrame Transaction
- 트랜잭션 서비스는 Spring 트랜잭션 서비스를 채택
- 트랜잭션 서비스 종류
  - DataSource Transaction Service
  - JTA Transaction Service
  - JPA Transaction Service에 대해서 설명한다.
- 트랜잭션 활용 방법
  - XML 설정 및 Annotation을 통해 활용할 수 있는 Declaration Transaction Management
  - 프로그램에서 직접 API를 호출하여 쓸 수 있도록 하는 Programmatic Transaction Management
  
### Transaction Service
- DataSource Transaction Service
  - DataSource를 사용하여 Local Transaction을 관리 할 수 있다.
  - Configuration
  <img src="https://cys779988.github.io/assets/img/egov-59.PNG">  
  <img src="https://cys779988.github.io/assets/img/egov-60.PNG">  

- JTA Transaction Service
  - JTA를 이용하여 Global Transation관리를 할 수 있도록 지원한다.
  - Configuration
  <img src="https://cys779988.github.io/assets/img/egov-61.PNG">  

- JPA Transaction Service
  - JPA Transaction 서비스는 JPA EntityManagerFactory를 이용하여 트랜잭션을 관리한다.
JpaTransactionManager는 EntityManagerFactory에 의존성을 가지고 있으므로 반드시
EntityManagerFactory 설정과 함께 정의되어야 한다. 아래에서 예를 들어서 설정 방법을 설명한다. 사용법은
DataSource Transaction Service와 동일하다.
  <img src="https://cys779988.github.io/assets/img/egov-62.PNG">  


### Declarative Transaction Management
- 코드에서 직접적으로 Transaction 처리하지 않고, 선언적으로 Transaction을 관리할 수 있다. Annotation을 이용한 Transaction 관리, XML 정의를 이용한 Transaction 관리를 지원한다.
  - Configuration
  <img src="https://cys779988.github.io/assets/img/egov-63.PNG">  
  
- Configuration Transaction Management
  - XML 정의 설정을 이용해서 Transaction을 관리
  - Configuration
  <img src="https://cys779988.github.io/assets/img/egov-64.PNG">  
  
  - \<tx:method\> 상세 속성 정보
    
  속성 | 설명 | 사용예
  ---- | ---- | ----
  name | 메소드명 기술. 와일드카드 사용 가능함 | Name=“find*”
  isolation | Transaction의 isolation Level 정의하는 요소 | Isolation=“DEFAULT”
  no-rollback-for | 정의된 Exception 목록에 대해서는 rollback을 수행하지 않음 | No-rollbackfor=“NoRolBackTx”
  propagation | Transaction의 propagation 유형을 정의하기 위한 요소 | propagation=“REQUIRED”
  read-only | 해당 Transaction을 읽기 전용 모드로 처리(Default=false) | read-only=“true”
  rollback-for | 정의된 Exception 목록에 대해서는 rollback 수행 | rollback-for=RoleBackTx”
  timeout | 지정한 시간 내에 해당 메소드 수행이 완료되지 않은 경우 rollback 수행 | timeout=“10”
  

  <img src="https://cys779988.github.io/assets/img/egov-65.PNG">  
  
