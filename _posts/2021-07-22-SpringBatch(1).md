---
title:  "spring batch(1) 기본개념"
excerpt: spring batch(1)
categories:
  - spring
---

## Spring Batch 기능
- 스프링 배치는 당연히 Spring core 모듈에 대한 의존관계를 가지고 있으며 그에 따라 POJO 기반의 개발, 유연한 설정법, AOP 적용 등 기존 스프링의 장점을 그대로 이어받았다.

#### 1. 각종 읽기와 쓰기 기능의 구성요소를 같은 인터페이스들로 추상화 시키고 있고 그에 맞는 기본 구현 클래스를 제공  
- DB, 플랫파일, XML 파일, JMS(Java Message Service) 등의 여러가지 출처와 형태로 연결되는 데이터들을 ItemReader와 ItemWriter라는 인터페이스를 통해 동일한 방식으로 접근할 수 있다.
- 인터페이스는 각각의 요소들의 역할 구분을 분명히 하고, 데이터 형태의 변화에 대응하는 유연성을 높일 수 있다.
- 부분적인 모듈의 테스트도 쉽게 할 수 있다.

#### 2. 대용량 조회 처리를 위해 커서기반이나 Driving query기반의 조회방식 지원
- 전자가 JdbcCursorItemReader나 HibernateCursorItemReader 클래스로 제공되고, 후자가 DrivingQueryItemReader, IbatisDrivingQueryItemReader 클래스를 통해 지원된다.

#### 3. Stream방식의 파일 처리를 더욱 편리하게 할 수 있는 API를 제공
- 열고 닫는 것이 필요한 자원에는 공통적으로 ItemStream이라는 인터페이스를 통해 이를 지원한다.
- StAX(Streaming API for XML) 기반의 API로 대용량 XML파일 처리를 쉽게한다.
- Spring Web Service 프레임워크의 일부인 Spring OXM(Object/XML Mapping framework)과 연결해서 특정 라이브러리에 종속적이지 않은 XML 파일 처리도 가능하다.

#### 4. 배치에 적합한 트랜잭션 처리를 위해 주기적인 commit 방식 지원
- SimpleStepFactoryBean 클래스를 통해 설정할 수 있는 commitInterval이라는 속성이 한번의 트랜잭션에서 처리될 수
- 데이터 특성이나 업무규칙에 따라 적절한 값을 트랜잭션 단위로 묶이는 건수를 Java 코드를 수정하지 않고도 설정으로 지정할 수 있다.

#### 5. 배치작업의 재시도, 재시작, 건너뛰기 등의 정책을 설정으로 적용함
- 스프링 배치에서는 네트워크 이상 같은 일시적인 장애로 인한 처리 실패를 대비해서 배치작업을 재시도하는 횟수를 지정할 수 있다.
- 재시작 시에는 이미 성공한 단계에 대해서는 건너뛸 수 있는 기능이 있고, ExecutionContext라는 저장 공간을 통해 이전 처리 때 생성한 정보를 활용할 수 있다.

#### 6. 유연한 Exception 처리
- Skip 해야 할 Exception, 또는 그 Skip 할 수 있는 횟수 등을 정의해서 업무 로직의 특성에 맞는 Exception 처리를 복잡한 try catch문 없이 설정만으로 할 수 있다.

#### 7. 각종 이벤트 처리
- Batch Job과 그것을 구성하는 단계 또는 트랜잭션의 단위 별로 그 시작과 끝, 에러 발생 시점에 호출되는 Listener 인터페이스가 정의
- Listener들을 구현해서 코딩하고 설정으로 연결하면 공통적인 작업들을 Batch 처리 코드와 섞이지 않게 넣어서 실행시킬 수 있다.

#### 8. Commit 개수, Rollback 개수, 재시도 횟수 등 배치실행 통계정보를 제공
- 개발자가 작성하는 코드에서 별도로 이를 위한 변수를 할당하고 로깅하는 코드를 집어 넣어줘야 필요가 없어졌다.

#### 9. 다양한 실행방법을 선택
- Spring의 Quartz 지원 클래스를 이용해서 스케줄링 기능을 Spring의 설정방식으로 지정할 수 있다.
- 수동으로 command line이나 JMX(Java Management Extensions) 콘솔 등을 통한 실행도 가능하다. 실행방식도 동기, 비동기-병렬 실행이 가능

#### 10. 추가 코딩없이 설정만으로 기존 모듈 활용
- 기존 모듈 중 배치에서 하는 것과 동일한 작업을 하는 코드가 있다면 따로 스프링배치의 인터페이스로 감싸서 코딩할 필요없이 applicationContext에서 설정만으로 연결이 가능하다.
- 기존모듈을 활용하면서도 스프링배치가 제공하는 이벤트처리, 실행정보 제공 등의 장점을 누릴 수 있다.

#### 11. iBatis나 Hibernate를 사용해서 DB접근모듈을 개발
#### 12. Validation 기능을 SpringValidator와 apache commons validator를 사용해서 구현

## 스프링 배치 구조

<img src="https://cys779988.github.io/assets/img/springbatch(1).PNG">
