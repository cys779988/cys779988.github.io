---
title:  "spring batch(4) Scope & Job Parameter"
excerpt: spring batch(4)
categories:
  - spring
---

## JobParameter와 Scope
- Spring Batch는 외부, 내부에서 파라미터를 받아 여러 Batch 컴포넌트에서 사용할 수 있게 지원. 이 파라미터를 Job Parameter라고 함
- Job Parameter를 사용하기 위해 Spring Batch 전용 Scope를 선언해야함.   ```@StepScope```  와   ```@JobScope```   2가지로 나눔

  
```@Value("#{jobParameter[파라미터명]}") ```  

- jobParameter 외에도 jobExecutionContext, stepExecutionContext 등도 SpEL로 사용할 수 있음
-   ```@JobScope```  에서는 stepExecutionContext는 사용할 수 없고 jobParameters와 jobExecutionContext만 사용할 수 있음

- JobScope
  
```java

	@Bean
	public Job scopeJob() {
		return jobBuilderFactory.get("scopeJob")
				.start(scopeStep1(null))
				.next(scopeStep2())
				.build();
	}
	
	@Bean
	@JobScope
	public Step scopeStep1(@Value("#{jobParameter[requestDate]}") String requestDate) {
		return stepBuilderFactory.get("scopeStep1")
				.tasklet((contribution, chunkCount) -> {
					log.info(">>>>> This is scopeStep1");
					log.info(">>>>> requestDate = {}", requestDate);
					return RepeatStatus.FINISHED;
				})
				.build();
	}
	
```  

- StepScope
  
```java

	@Bean
	private Step scopeStep2() {
		// TODO Auto-generated method stub
		return stepBuilderFactory.get("scopeStep2")
				.tasklet(scopeStep2Tasklet(null))
				.build();
	}

	@Bean
	@StepScope
	public Tasklet scopeStep2Tasklet(@Value("#{jobParameter[requestDate]}") String requestDate) {
		return (contribution, chunkCount) -> {
					log.info(">>>>> This is scopeStep1");
					log.info(">>>>> requestDate = {}", requestDate);
					return RepeatStatus.FINISHED;
		};
	}
```  

- @JobScope는 Step 선언문에서 사용 가능하고, @StepScope는 Tasklet이나 ItemReader, ItemWriter, ItemProcessor에서 사용 가능
- Job Parameter의 타입으로 사용할 수 있는 것은 Double, Long, Date, String
- 예제코드에서 Job Parameter에 null을 할당하고 있는데 이는 Job Parameter의 할당을 어플리케이션 실행시에 하지 않음을 뜻함

## @StepScope & @JobScope
- Spring Batch는   ```@StepScope```  와   ```@JobScope```  라는 특별한 Bean Scope를 지원함
- Spring Bean의 기본 Scope는 singleton인데 Spring Batch 컴포넌트(Tasklet, ItemReader, ItemWriter, ItemProcessor 등)에   ```@StepScope```  를 사용하게 되면 
Spring Batch가 Spring 컨테이너를 통해 지정된 Step의 실행시점에 해당 컴포넌트를 Spring Bean으로 생성함.


```java
	@Bean
	@StepScope
	public ListItemReader<Integer> simpleWriterReader() {
		List<Integer> items = new ArrayList<>();
		
		for (int i = 0; i < 100; i++) {
			items.add(i);
		}
		return new ListItemReader<>(items);
	}
```  

-   ```@JobScope```  는 Job 실행시점에 Bean이 생성됨. 즉, Bean의 생성 시점을 지정된 Scope가 실행되는 시점으로 지연시킴
- JobScope, StepScope는 Job이 실행되고 끝날 때, Step이 실행되고 끝날 때 생성/삭제가 이루어짐
  
이렇게 Bean의 생성시점을 어플리케이션 실행시점이 아닌, Step 혹은 Job의 실행시점으로 지연시킴으로써 장점
  
1. JobParameter의 Late Binding 이 가능함
  
Job Parameter를 StepContext 또는 JobExecutionContext 레벨에서 할당시킬 수 있음. 꼭 Application이 실행되는 시점이 아니더라도 Controller나 Service 같은 비즈니스 로직 처리단계에서 Job Parameter를 할당시킬 수 있음

2. 동일한 컴포넌트를 병렬 혹은 동시에 사용할 때 유용
  
Step 안에 Tasklet이 있고 이 Tasklet은 멤버변수와 이 멤버변수를 변경하는 로직이 있다고 가정하면 이 경우   ```@StepScope```   없이 Step을 병렬로 실행시키게 되면 서로 다른 Step에서 하나의 Tasklet을 두고 마구잡이로 상태를 변경하려고 함. 하지만   ```@StepScope```   이 있다면 각각의 Step에서 별도의 Tasklet을 생성하고 관리하기 때문에 서로의 상태를 침범할 일이 없어짐

## Job Parameter 오해
  
Job Parameter는   ```@Value```  를 통해서 가능해서 여러가지 오해가 발생하는데 Job Parameter는 Step이나 Tasklet, Reader 등 Batch 컴포넌트 Bean의 생성 시점에 호출할 수 있지만 정확히는 Scope Bean을 생성할 때만 가능.  
즉   ```@StepScope```  ,   ```@JobScope```  로 Bean을 생성할때만 Job Parameter가 생성되기 때문에 사용할 수 있음
