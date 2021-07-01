---
title:  "spring boot Application Event"
excerpt: spring boot Application Event
categories:
  - spring
---


## ApplicationEventPublisher
- Spring ApplicationContext 클래스는 ApplicationEventPublisher 인터페이스가 이미 구현되어 있음
- ApplicationEventPublisher를 이용하면 쉽게 이벤트를 발생시키고 처리할 수 있음

  
```java
1. Event 정의
2. Event 발생 - ApplicationEventPublisher.publishEvent()
3. @EventListner 어노테이션 적용된 메서드 @Order 순서대로 실행
이벤트 비동기 처리시에는 @Async 어노테이션을 이벤트 리스너에 적용
```  

  
```java
@Getter
@Setter
@ToString
@NoArgsConstructor
public class MemberDto {
	private Long id;
	
	@Builder
	public MemberDto(Long id) {
		this.id = id;
	}
}
```  

  
```java
@Component
public class ApplicationRunner implements org.springframework.boot.ApplicationRunner{
	@Autowired
	ApplicationEventPublisher applicationEventPublisher;
	
	@Override
	public void run(ApplicationArguments args) throws Exception {
		// TODO Auto-generated method stub
		MemberDto member =  MemberDto.builder()
					.id(Long.parseLong("2143"))
					.build();
		System.out.println("event start");
		applicationEventPublisher.publishEvent(member);
		System.out.println("event end");
	}
}
```  
  
  
```java
@Component
public class SampleListener{

	@EventListener
	@Order(value = 2)
	public void MemberFirstEvent(MemberDto member) {
		System.out.println("안녕하세요1! " + member.getId());
	}

	@EventListener
	@Order(value = 1)
	public void MemberSecondEvent(MemberDto member) {
		System.out.println("안녕하세요2! " + member.getId());
	}
	
	@EventListener
	@Order(value = 3)
	public void MemberThirdEvent(MemberDto member) {
		System.out.println("안녕하세요3! " + member.getId());
	}
}
```  

  
```java
@EnableJpaAuditing
@SpringBootApplication
public class DemoApplication {

	public static void main(String[] args) {
		SpringApplication springApplication = new SpringApplication(DemoApplication.class);
		springApplication.addListeners((ApplicationListener<ApplicationStartingEvent>) applicationEvent ->{
			System.out.println("ApplicationStartingEvent 테스트");
		});
		springApplication.run(args);
	}
}
```  

<img src="https://cys779988.github.io/assets/img/applicationEvent.PNG">
