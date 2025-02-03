---
title:  "spring AOP"
excerpt: spring AOP
categories:
  - spring
---

## AOP
- 객체지향 프로그래밍을 보완하는 개념으로 애플리케이션을 객체지향적으로 모듈화 하여 작성하더라도 다수의 객체들에 분산되어 중복적으로 존재하는 공통 관심사가 여전히 존재한다. AOP는 이를 횡단관심으로 분리하여 핵심관심과 엮어서 처리할 수 있는 방법을 제공한다.
- 로깅, 보안, 트랜잭션 등의 공통적인 기능의 활용을 기존의 비즈니스 로직에 영향을 주지 않고 모듈화 처리를 지원하는 프로그래밍 기법

<img src="https://cys779988.github.io/assets/img/springAop-1.PNG">  

## 장점
- 중복 코드의 제거 : 횡단 관심(CrossCutting Concerns)을 여러 모듈에 반복적으로 기술되는 현상을 방지
- 비즈니스 로직의 가독성 향상 : 핵심기능 코드로부터 횡단 관심 코드를 분리함으로써 비즈니스 로직의 가독성 향상
- 생산성 향상 : 비즈니스 로직의 독립으로 인한 개발의 집중력을 높임
- 재사용성 향상 : 횡단 관심 코드는 여러 모듈에서 재사용될 수 있음
- 변경 용이성 증대 : 횡단 관심 코드가 하나의 모듈로 관리되기 때문에 이에 대한 변경 발생시 용이하게 수행할 수 있음


## 주요개념
  
용어 | 설명
---- | ----
Aspect | 애플리케이션이 가지고 있어야 할 로직과 그것을 실행해야 하는 지점을 정의한 것(Advice와 Pointcut의 조합).
Advice | Aspect)의 실제 구현체로 결합점에 삽입되어 동작할 수 있는 코드. JoinPoint와 결합하여 동작하는 시점에 따라 before advice, after advice, around advice 타입으로 구분.
Joinpoint | 횡단 관심 모듈이 삽입되어 동작할 수 있는 실행 가능한 특정 위치(Advice를 적용하는 지점을 의미). 스프링은 method 결합점만 제공.
Pointcut | 어떤 클래스의 어느 JoinPoint를 사용할 것인지를 결정하는 선택 기능(Advice가 적용되는 대상을 지정)을 의미. 패키지이름/클래스이름/메서드이름을 정규식으로 지정하여 사용.
Target | Advice가 적용되는 클래스를 의미(Aspect를 정의하는 곳)
Weaving | Pointcut에 의해서 결정된 JoinPoint에 지정된 Advice를 삽입하는 과정(Advice를 주기능에 적용하는 것을 의미).

<img src="https://cys779988.github.io/assets/img/springAop-2.PNG">  
<img src="https://cys779988.github.io/assets/img/springAop-3.PNG">  

## Aspect 실행 시점 지정
  
애너테이션 | 기능
---- | ---- 
@Before (이전) | 어드바이스 타겟 메소드가 호출되기 전에 어드바이스 기능을 수행
@After (이후) | 타겟 메소드의 결과에 관계없이(즉 성공, 예외 관계없이) 타겟 메소드가 완료 되면 어드바이스 기능을 수행
@AfterReturning (정상적 반환 이후) | 타겟 메소드가 성공적으로 결과값을 반환 후에 어드바이스 기능을 수행
@AfterThrowing (예외 발생 이후) | 타겟 메소드가 수행 중 예외를 던지게 되면 어드바이스 기능을 수행
@Around (메소드 실행 전후) | 어드바이스가 타겟 메소드를 감싸서 타겟 메소드 호출 전과 후에 어드바이스 기능을 수행

## 스프링에서 AOP기능 구현 예제

#### 스프링 AOP 의존성 추가
  
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```  

#### 경로지정 방식
  
```java
@Component
@Aspect
public class PerfAspect {
    @Around("execution(* com.study..*.EventService.*(..))")
    public Object logPerf(ProceedingJoinPoint pjp) throws Throwable {
        long begin = System.currentTimeMillis();
        Object retVal = pjp.proceed(); // 메서드 호출 자체를 감싼다
        System.out.println(System.currentTimeMillis() - begin);
        return retVal;
    }
}
```  
@Around 어노테이션은 타겟 메서드를 감싸서 특정 Advice를 실행한다는 의미이다. 즉, `execution(* com.study..*.EventService.*(..))`는 com.study 아래 패키지 경로의 EventService 객체의 모든 메서드에 이 Aspect를 적용하겠다는 것을 의미한다.

#### 애너테이션 지정 방식

```java
@Component
@Aspect
public class PerfAspect {
    @Around("@annotation(PerLogging)")
    public Object logPerf(ProceedingJoinPoint pjp) throws Throwable{
        long begin = System.currentTimeMillis();
        Object retVal = pjp.proceed();
        System.out.println(System.currentTimeMillis() - begin);
        return retVal;
    }
}
```  

  
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.CLASS)
public @interface PerLogging {
}
```  
  
```java
@Component
public class SimpleEventService implements EventService {

    @PerLogging
    @Override
    public void createEvent() {
        try {
            Thread.sleep(1000);
        } catch(InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Created an event");
    }

    @Override
    public void publishEvent() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e){
            e.printStackTrace();;
        }
        System.out.println("Published an event");
    }

    @PerLogging
    @Override
    public void deleteEvent() {
        System.out.println("Delete an event");
    }
}
```  

#### 빈의 모든 메서드에 지정
  
```java
@Component
@Aspect
public class PerfAspect {
    @Around("bean(simpleEventService)")
    public Object logPerf(ProceedingJoinPoint pjp) throws Throwable{
        long begin = System.currentTimeMillis();
        Object retVal = pjp.proceed();
        System.out.println(System.currentTimeMillis() - begin);
        return retVal;
    }
}
```  

  
```java
@Component
public class SimpleEventService implements EventService {

    @Override
    public void createEvent() {
        try {
            Thread.sleep(1000);
        } catch(InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Created an event");
    }

    @Override
    public void publishEvent() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e){
            e.printStackTrace();;
        }
        System.out.println("Published an event");
    }
    
    @Override
    public void deleteEvent() {
        System.out.println("Delete an event");
    }
}
```  
  
```java
@Service
public class AppRunner implements ApplicationRunner {

    @Autowired
    EventService eventService;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        eventService.createEvent();
        eventService.publishEvent();
        eventService.deleteEvent();
    }
}
```  

#### 표현식을 변수로 활용
@Pointcut 애너테이션은 Aspect에서 변수와 같이 재사용 가능한 포인트컷을 정의할 수 있다.
  
```java
@Aspect
public class Performance {

    @Pointcut("execution(* com.study.board.BoardService.getBoards(..))")
    public void getBoards(){}

    @Pointcut("execution(* com.study.user.UserService.getUsers(..))")
    public void getUsers(){}

    @Around("getBoards() || getUsers()")
    public Object calculatePerformanceTime(ProceedingJoinPoint proceedingJoinPoint) {
        Object result = null;
        try {
            long start = System.currentTimeMillis();
            result = proceedingJoinPoint.proceed();
            long end = System.currentTimeMillis();

            System.out.println(end - start);
        } catch (Throwable throwable) {
            System.out.println("exception");
        }
        return result;
    }
}
```  

#### 타겟 메서드의 인자를 사용
  
```java
@Aspect
@Component
public class UserAspect {

    @Autowired
    private UserRepository userRepository;

    @Pointcut("execution(* com.study.user.UserService.update(*)) && args(user)")
    public void updateUser(User user){}

    @AfterReturning("updateUser(user)")
    public void saveHistory(User user){
    	userRepository.save(new History(user.getIdx()));
    }
}
```  

`args(user)` 표현식을 통해 타겟 메서드의 인자와 어드바이스의 인자가 매칭된다. 즉, updateUser라는 포인트컷이 user라는 인자를 사용하도록 `args(user)` 표현식으로 지정한 것 이다.