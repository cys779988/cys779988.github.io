---
title:  "eGovFrame ID Generation, Logging"
excerpt: eGovFrame ID Generation와 logging 기능
categories:
  - eGovFrame
---

### ID Generation
다양한 형식의 ID 구조 및 다양한 방식의 ID 생성 알고리즘을 제공하여 시스템에서 사용하는 ID(Identifier)를 생성하는 서비스  
<img src="https://cys779988.github.io/assets/img/egov-29.PNG">  

### ID Generation 주요기능
- UUID(Universal Unique Identifier) 생성
- Sequence ID 생성  
순차적으로 증가 또는 감소하는 Sequence ID를 생성한다. 시스템에서는 다수의 Sequence ID가 사용되므로, 각각의 Sequence ID는 구별된다. 시스템의 재시작 시에도 Sequence ID는 마지막 생성된 ID의 다음 ID를 생성한다.
  - Sequence ID 생성 : DB의 SEQUENCE를 활용하여 ID를 생성한다.
  - Table ID 생성 : 키제공을 위한 테이블을 지정하여 ID를 생성한다.

### Logging
- 시스템의 개발이나 운용 시 발생할 수 있는 어플리케이션 내부정보에 대해서 시스템의 외부 저장소에 기록하여 시스템의 상황을 쉽게 파악할 수 있게 지원하는 서비스
- 많은 개발자가 Log을 출력하기 위해 일반적으로 사용하는 방식은 System.out.println()임. 하지만 이 방식은 간편한 반면에 다음과 같은 이유로 권장하지 않음.
  - 콘솔 로그를 출력 파일로 리다이렉트 할 지라도, 어플리케이션 서버가 재 시작할 때 파일이 overwrite될 수도 있음
  - 개발/테스팅 시점에만 System.out.println()을 사용하고 운영으로 이관하기 전에 삭제하는 것은 좋은 방법이 아님
  - System.out.println() 호출은 디스크 I/O동안 동기화(synchronized)처리가 되므로 시스템의 throughput을 떨어뜨림
  - 기본적으로 stack trace 결과는 콘솔에 남는다. 하지만 시스템 운영 중 콘솔을 통해 Exception을 추적하는 것은 바람직하지 못함
  
### Logging 주요기능
- 로깅 환경 설정 지원
  - 서브 시스템 별 상세한 로그 정책 부여
  - 다양한 형식(날짜형식, 시간형식 등)의 로그 메시지 형태 지정
  - 다양한 매체(File, DBMS, Message, Mail 등)에 대한 기록 기능 설정
- 로그 기록
  - 레벨(debug, info, warn, error 등)별로 로그를 기록
  
### Log4j 환경설정하는 방법
- 프로그래밍내에서 직접 설정하는 방법
- 설정 파일을 사용하는 방법

### 중요 컴포넌트 설명
  
컴포넌트 | 설명
---- | ----
Logger | 로그의 주체 (로그 파일을 작성하는 클래스) – 설정을 제외한 거의 모든 로깅 기능이 이를 통해 처리됨. 어플리케이션 별로 사용할 로거(로거명 기반)를 정의하고 이에 대해 로그레벨과 Appender를 지정할 수 있음.
Appender | 로그를 출력하는 위치를 의미하며, Log4J API문서의 XXXAppender로 끝나는 클래스들의 이름을 보면, 출력위치를 어느정도 짐작할 수 있음.
Layout | Appender의 출력포맷 - 일자, 시간, 클래스명등 여러가지 정보를 선택하여 로그정보내용으로 지정할 수 있음. 
  
### 로그레벨 지정하기
- log4j에서는 기본적으로 debug, info, warn, error, fatal의 다섯 가지 로그레벨이 있음
- (ERROR \> WARN \> INFO \> DEBUG \> TRACE)
  
로그 레벨 | 설명
---- | ----
Error | 요청을 처리하는 중 문제가 발생한 상태
Warn | 처리 가능한 문제이지만 향후 시스템 에러의 원인이 될 수 있는 경고성 메시지
Info | 로그인, 상태변경과 같은 정보성 메시지
Debug | 개발시 디버그 용도로 사용할 메시지
Trace | 디버그 레벨이 너무 광범위한 것을 해결하기 위한 좀 더 상세한 상태
  
### Appender
log4j 는 콘솔, 파일, DB, socket, message, mail 등 다양한 로그 출력 대상과 방법을 지원하는데, 이에 대해 log4j 의 Appender 로 정의
  
Appender | 설명
---- | ----
ConsoleAppender | 콘솔화면으로 출력하기 위한 appender
FileAppender | 로깅을 파일에 하고 싶을 때 사용
RollingFileAppender | 파일의 크기 또는 파일백업인덱스 등의 지정을 통해서 특정크기 이상 파일의 크기가 커지게 되면 기존파일을 백업파일로 바꾸고, 다시 처음부터 로깅을 시작함
JDBCAppender | DB에 로그를 출력하기 위한 Appender로 하위에 Driver, URL, User, Password, Sql과 같은 parameter를 정의할 수 있음 EgovConnectionFactory 빈 설정이 필요함
  
