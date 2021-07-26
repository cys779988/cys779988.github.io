---
title:  "spring batch(3) Job Flow"
excerpt: spring batch(3)
categories:
  - spring
---

## next

- next()는 순차적으로 Step을 연결시킬 때 사용

  
```java

@Slf4j
@Configuration
@RequiredArgsConstructor
public class StepNextJobConfiguration {

	private final JobBuilderFactory jobBuilderFactory;
	private final StepBuilderFactory stepBuilderFactory;
	
	@Bean
	public Job stepNextJob() {
		return jobBuilderFactory.get("stepNextJob")
				.start(step1())
				.next(step2())
				.next(step3())
				.build();
	}

	@Bean
	private Step step1() {
		// TODO Auto-generated method stub
		return stepBuilderFactory.get("step1")
				.tasklet((contribution, chunkContext) -> {
					log.info(">>>>>>> This is Step1");
					return RepeatStatus.FINISHED;
				})
				.build();
	}

	@Bean
	private Step step2() {
		// TODO Auto-generated method stub
		return stepBuilderFactory.get("step2")
				.tasklet((contribution, chunkContext) -> {
					log.info(">>>>>>> This is Step2");
					return RepeatStatus.FINISHED;
				})
				.build();
	}

	@Bean
	private Step step3() {
		// TODO Auto-generated method stub
		return stepBuilderFactory.get("step3")
				.tasklet((contribution, chunkContext) -> {
					log.info(">>>>>>> This is Step3");
					return RepeatStatus.FINISHED;
				})
				.build();
	}
}
```  

#### 지정한 Batch Job만 실행되도록 설정

  
```
//src/main/resources/application.properties

spring.batch.job.names: ${job.name:NONE}
```  

- Spring Batch가 실행될 때, Program arguments로 job.name 값이 넘어오면 해당 값과 일치하는 Job만 실행
- 코드의 의미는 job.name이 있으면 job.name 값을 할당하고 없으면 NONE을 할당하겠다는 의미
- spring.batch.job.names에 NONE이 할당되면 어떤 배치도 실행하지 않겠다는 의미 (값이 없을 때 모든 배치가 실행되지 않도록 막는 역할)

<img src="https://cys779988.github.io/assets/img/springbatch-4.PNG">

실제 운영 환경에서는   ```java -jar batch-application.jar --job.name=simpleJob```   과 같이 배치를 실행함

## 조건별 흐름 제어(flow)
- next()가 순차적으로 Step의 순서를 제어할 때 앞의 step에서 오류가 발생했을 때 나머지 뒤에 있는 step들은 실행되지 못함
- 상황에 따라 조건별로 Step을 사용하기 위해 흐름 제어를 사용함

  
```java
@Slf4j
@Configuration
@RequiredArgsConstructor
public class StepNextConditionalJobConfiguration {

	private final JobBuilderFactory jobBuilderFactory;
	private final StepBuilderFactory stepBuilderFactory;
	
	@Bean
	public Job stepNextConditionalJob() {
		return jobBuilderFactory.get("stepNextConditionalJob")
				.start(conditionalJobStep1())
					.on("FAILED")
					.to(conditionalJobStep3())
					.on("*")
					.end()
				.from(conditionalJobStep1())
					.on("*")
					.to(conditionalJobStep2())
					.next(conditionalJobStep3())
					.on("*")
					.end()
				.end()
				.build();
	}


	@Bean
	public Step conditionalJobStep1() {
		// TODO Auto-generated method stub
		return stepBuilderFactory.get("step1")
				.tasklet((contribution, chunkContext) -> {
					log.info(">>>> This is stepNextConditionalJob Step1");
					contribution.setExitStatus(ExitStatus.FAILED);
					return RepeatStatus.FINISHED;
				})
				.build();
	}
	
	@Bean
	public Step conditionalJobStep2() {
		// TODO Auto-generated method stub
		return stepBuilderFactory.get("conditionalJobStep2")
				.tasklet((contribution, chunkContext) -> {
					log.info(">>>> This is stepNextConditionalJob Step2");
					return RepeatStatus.FINISHED;
				})
				.build();
	}
	
	@Bean
	public Step conditionalJobStep3() {
		// TODO Auto-generated method stub
		return stepBuilderFactory.get("conditionalJobStep3")
				.tasklet((contribution, chunkContext) -> {
					log.info(">>>> This is stepNextConditionalJob Step3");
					return RepeatStatus.FINISHED;
				})
				.build();
	}
}
```  

#### on()
- 캐치할 ExitStatus 지정
-   ```*```  일 경우 모든 ExitStatus가 지정됨

#### to()
- 다음으로 이동할 Step 지정

#### from()
- 일종의 이벤트 리스너 역할
- 상태값을 보고 일치하는 상태라면   ```to()```  에 포함된   ```step```  을 호출함
- step1의 이벤트 캐치가 FAILED로 되어있는 상태에서 추가로 이벤트 캐치하려면   ```from()```  을 써야만 함

#### end()
- end는 FlowBuilder를 반환하는 end와 FlowBuilder를 종료하는 end 2개가 있음
-   ```on("*")```   뒤에 있는 end는 FlowBuilder를 반환하는 end
-   ```build()```   앞에 있는 end는 FlowBuilder를 종료하는 end
- FlowBuilder를 반환하는 end 사용시 계속해서   ```from```  을 이어갈 수 있음


#### Batch Status vs. Exit Status
- BatchStatus는 Job 또는 Step의 실행결과를 Spring에서 기록할 때 사용하는 Enum
- BatchStatus로 사용되는 값은 COMPLETED, STARTING, STARTED, STOPPING, STOPPED, FAILED, ABANDONED, UNKNOWN
- on() 메서드가 참조하는 값은 Step의 ExitStatus(Step의 실행 후 상태)
- Spring Batch는 기본적으로 ExitStatus의 exitCode는 Step의 BatchStatus와 같도록 설정돼 있음

#### ExitCode 커스터마이징

  
```java
.start(step1())
    .on("FAILED")
    .end()
.from(step1())
    .on("COMPLETED WITH SKIPS")
    .to(errorPrint1())
    .end()
.from(step1())
    .on("*")
    .to(step2())
    .end()
```  

  
```java
public class SkipCheckingListener extends StepExecutionListenerSupport {

    public ExitStatus afterStep(StepExecution stepExecution) {
        String exitCode = stepExecution.getExitStatus().getExitCode();
        if (!exitCode.equals(ExitStatus.FAILED.getExitCode()) && 
              stepExecution.getSkipCount() > 0) {
            return new ExitStatus("COMPLETED WITH SKIPS");
        }
        else {
            return null;
        }
    }
}
```  

- StepExecutionListener 에서 먼저 Step이 성공적으로 수행되었는지 확인하고, StepExecution의 skip 횟수가 0보다 클 경우   ```COMPLETED WITH SKIPS```  의 exitCode를 갖는 ExitStatus를 반환

## Decide
- 위 방식의 문제점
  - Step이 담당하는 역할이 2개 이상이 됨, 실제 해당 Step이 처리해야할 로직외에도 분기처리를 시키기 위해 ExitStatus 조작이 필요함
  - 다양한 분기 로직 처리의 어려움, ExitStatus를 커스텀하게 고치기 위해서는 Listener를 생성하고 Job Flow에 등록하는 번거로움이 존재함
- JobExecutionDecider는 Step 들의 flow 속에서 분기만 담당


