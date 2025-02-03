---
title:  "Transaction"
excerpt: Spring Transaction
categories:
  - spring
---


## 트랜잭션 ACID 특성
트랜잭션(Transaction)은 데이터베이스에서 여러 작업을 하나의 논리적인 단위로 묶어 실행하는 개념. 트랜잭션이 ACID 특성을 가지는 이유는 데이터의 무결성과 일관성을 보장하기 위함.

### Atomicity (원자성)

- 트랜잭션 내의 작업은 모두 성공하거나 모두 실패해야 한다.
- 중간에 실패할 경우, 모든 변경 사항을 롤백(Rollback)하여 데이터베이스가 일관된 상태를 유지해야 합니다.
- 예시: 은행 계좌 이체에서 A 계좌에서 100원을 빼고 B 계좌에 100원을 넣는 과정에서 하나라도 실패하면 전체가 롤백되어야 한다.

### Consistency (일관성)

- 트랜잭션이 실행되기 전과 후에 데이터베이스는 항상 일관된 상태를 유지해야 한다.
- 트랜잭션이 완료되면 모든 데이터는 정의된 무결성 제약 조건을 만족해야 한다.
- 예시: 한 학생이 수강 신청을 했을 때, 강의의 정원이 줄어들어야 하며, 반대로 취소하면 정원이 증가해야 한다.

### Isolation (고립성)

- 여러 트랜잭션이 동시에 실행될 때 서로 간섭하지 않도록 해야 한다.
- 격리 수준(Isolation Level)을 조절하여 데이터 정합성을 보장할 수 있다.
- 예시: 두 개의 트랜잭션이 동일한 제품의 재고를 줄이는 경우, 한 트랜잭션이 완료되기 전에 다른 트랜잭션이 영향을 주면 안 된다.

### Durability (지속성)

- 트랜잭션이 성공적으로 완료되면, 시스템 장애가 발생하더라도 데이터는 영구적으로 저장되어야 한다.
- 일반적으로 WAL(Write-Ahead Logging)과 같은 기법을 사용하여 장애 발생 시에도 데이터를 복구할 수 있어야 한다.
- 예시: 은행 계좌 잔액이 갱신된 후 시스템이 다운되더라도 변경된 데이터는 저장되어 있어야 합니다.

### ACID를 보장하기 위한 주요 기술
- Atomicity & Durability → 로그(Write-Ahead Logging), 체크포인트(Checkpoint)
- Consistency → 트랜잭션 제약 조건(Constraints), 데이터 무결성 보장
- Isolation → 격리 수준 설정 (Read Committed, Repeatable Read, Serializable 등)


## @Transactional 어노테이션 사용시 주의사항

### 1. 메서드가 public이어야 한다
- `@Transactional`은 스프링 AOP 기반으로 동작하기 때문에 프록시 객체가 생성된다.
- 내부에서 private 또는 protected 메서드를 호출하면 트랜잭션이 적용되지 않는다.

### 2. 동일 클래스 내부의 메서드 호출은 트랜잭션이 적용되지 않음
- 스프링의 기본 트랜잭션 관리 방식은 프록시 기반 AOP이기 때문에, 같은 클래스 내에서 `@Transactional`이 붙은 메서드를 호출하면 트랜잭션이 적용되지 않는다.

### 3. @Transactional이 인터페이스에서는 동작하지 않을 수도 있음
- JDK 동적 프록시(Proxy.newProxyInstance)는 인터페이스 기반으로 생성된다.
- 인터페이스가 존재하면 프록시가 인터페이스를 기준으로 생성되므로, 구현체의 `@Transactional`이 적용되지 않을 수도 있다.
- 해결 방법: proxyTargetClass = true로 설정하여 **CGLIB(클래스 기반 프록시)** 을 사용하도록 설정해야 한다.
`@EnableTransactionManagement(proxyTargetClass = true)`

### 4. @Transactional(readOnly = true)의 사용
- `@Transactional(readOnly = true)`를 사용하면 쓰기 연산이 차단되며, 데이터베이스의 성능이 최적화.
- 하지만 쓰기 연산을 하면 예외가 발생할 수 있으므로 주의해야 한다.

### 5. Checked Exception은 트랜잭션 롤백이 되지 않음
- 스프링의 기본 설정에서는 RuntimeException과 Error가 발생해야 자동 롤백됩니다.
- Checked Exception이 발생하면 기본적으로 트랜잭션이 커밋됩니다.


## 트랜잭션 전파(Propagation) 옵션
트랜잭션 전파(Propagation)는 하나의 트랜잭션이 실행 중일 때 새로운 트랜잭션을 어떻게 처리할지 결정하는 옵션.

  
옵션	| 설명
---- | ----
REQUIRED (기본값) | 기존 트랜잭션이 있으면 참여하고, 없으면 새 트랜잭션 생성
REQUIRES_NEW | 항상 새로운 트랜잭션을 생성, 기존 트랜잭션은 일시 정지
NESTED | 부모 트랜잭션 안에서 별도의 트랜잭션 실행 (Rollback 독립적 가능)
MANDATORY | 반드시 기존 트랜잭션이 있어야 실행 (없으면 예외 발생)
SUPPORTS | 트랜잭션이 있으면 참여하고, 없으면 트랜잭션 없이 실행
NOT_SUPPORTED | 트랜잭션 없이 실행, 기존 트랜잭션은 일시 정지
NEVER | 트랜잭션이 있으면 예외 발생
  

## 트랜잭션 격리 수준(Isolation Level)
- 여러 트랜잭션이 동시에 실행될 때 데이터의 일관성을 보장하기 위해 데이터베이스가 적용하는 규칙.
- SQL 표준에서는 총 4가지 격리 수준을 정의하고 있으며, 각 수준은 발생할 수 있는 문제를 방지하는 정도에 따라 나뉜다.
- 대부분의 애플리케이션에서는 Dirty Read(더티 리드)만 방지하면 충분한 경우가 많음.
  
격리 수준 (Isolation Level) | Dirty Read (더티 리드) | Non-Repeatable Read (반복 불가능한 읽기) | Phantom Read (팬텀 리드) | 특징
---- | ---- | ---- | ---- | ----
Read Uncommitted (읽기 미완료 허용) | 발생 가능 | 발생 가능 | 발생 가능 | 가장 낮은 격리 수준, 미확정 데이터 읽기 가능
Read Committed (읽기 완료 허용) | 방지 | 발생 가능 | 발생 가능 | 커밋된 데이터만 읽음
Repeatable Read (반복 가능 읽기) | 방지 | 방지 | 발생 가능 | 같은 트랜잭션 내 동일한 조회 보장
Serializable (직렬화) | 방지 | 방지 | 방지 | 가장 엄격한 격리 수준, 동시성 낮음


### DBMS별 트랜잭션 격리 수준

| DBMS | 기본 격리 수준 | 지원하는 격리 수준 |
| --- | --- | --- |
| MySQL (InnoDB) | Repeatable Read | Read Uncommitted, Read Committed, Repeatable Read, Serializable |
| Oracle | Read Committed | Read Committed, Serializable, Read-Only |
| PostgreSQL | Read Committed | Read Committed, Repeatable Read, Serializable |
| MongoDB | Read Committed 수준 (일관성 모델이 다름) | No explicit isolation levels (document-level isolation) |

- MySQL InnoDB 스토리지 엔진은 Repeatable Read가 기본값이며, 이를 유지하면서 Gap Lock + Next-Key Lock을 통해 팬텀 리드까지 방지 가능. 일반적인 Repeatable Read보다 강력하여 사실상 Serializable에 가까운 수준을 제공.
- PostgreSQL은 MVCC 기반으로 구현되어 Read Committed에서도 높은 일관성을 유지함.
- MongoDB는 RDBMS와 다른 격리 수준을 가짐. 기본적으로 Read Committed 수준의 일관성을 제공하지만, 트랜잭션을 지원하는 경우 Snapshot Isolation 수준을 제공 가능.