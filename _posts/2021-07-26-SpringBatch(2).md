---
title:  "spring batch(2) Batch Job 메타데이터"
excerpt: spring batch(2)
categories:
  - spring
---

## Spring Batch Metadata

<img src="https://cys779988.github.io/assets/img/springbatch-2.PNG">

참조 : https://docs.spring.io/spring-batch/docs/4.3.x/reference/html/schema-appendix.html#metaDataSchema

#### BATCH_JOB_INSTANCE
- Job이 실행되면 BATCH_JOB_INSTANCE에 새로운 row를 만듦. Job의 이름과 Job이 시작될 때 넘겨받은 파라미터를 Serialize(직렬화)해서 저장
- BATCH_JOB_INSTANCE 테이블은 Job Parameter(Spring Batch가 실행될 때 외부에서 받을 수 있는 파라미터)에 따라 생성되는 테이블
- 같은 Batch Job 이라도 Job Parameter가 다르면 BATCH_JOB_INSTANCE에는 기록되지만 Job Parameter가 같다면 기록되지 않음
  
Job의 flow(high level)
1. 시작
2. BATCH_JOB_INSTANCE에 동일한 JOB_NAME과 JOB_KEY를 가진 row가 있는지 확인. 찾으면 JobInstance must not already exist 에러 메시지 출력 후 종료
3. BATCH_JOB_INSTANCE에 row 생성
4. BATCH_JOB_EXECUTION에 BATCH_JOB_INSTANCE_ID를 가진 가장 마지막 row가 있는지 확인
5. 찾으면 Completed 상태가 아닐 경우에만 재시작. 그 외에는 에러 메시지 출력 후 종료
6. 못 찾으면 BATCH_JOB_EXECUTION에 row 생성
7. Step 실행
8. 종료



#### BATCH_JOB_EXECUTION
- Job 실행내용을 담고 있음. Job의 실패 + 성공 횟수만큼 row가 생성
- BATCH_JOB_INSTANCE와 BATCH_JOB_EXECUTION은 부모-자식 관계
- BATCH_JOB_EXECUTION은 자신의 부모 BATCH_JOB_INSTANCE가 성공/실패 했던 모든 내역을 가지고 있음
- Spring Batch는 동일한 Job Parameter로 성공한 기록이 있을때만 재수행이 안됨

  
BATCH_JOB_EXECUTION_PARAMS : Job 파라미터가 저장됨. IDENTIFYING 컬럼은 BATCH_JOB_INSTANCE.JOB_KEY에 포함될지 말지 결정함
  
BATCH_JOB_EXECUTION_CONTEXT : Job 안에 있는 컴포넌트들(tasklet, step 등)이 정보를 교환해야 할 때 JOB_EXECUTION_CONTEXT를 사용해 정보를 넣거나 빼낼 수 있음. 그런 정보들이 저장된 테이블

#### BATCH_STEP_EXECUTION
- Step 실행 내용을 담고 있음. JOB_EXECUTION과 마찬가지로 Step의 실패 + 성공 횟수만큼 row가 생성됨
  
BATCH_STEP_EXECUTION_CONTEXT : Step 안에 있는 컴포넌트들(reader, processor, writer 등)이 정보를 교환할 때 해당 테이블에 저장됨

