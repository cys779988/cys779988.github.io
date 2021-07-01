---
title:  "spring AOP"
excerpt: spring AOP기능
categories:
  - spring
---

### 여러가지 AOP관련 용어
  
용어 | 설명
---- | ----
aspect | 구현하고자 하는 보조 기능을 의미
advice | aspect의 실제 구현체(클래스)를 의미. 메서드 호출을 기준으로 여러 지점에서 실행됨
joinpoint | advice를 적용하는 지점을 의미. 스프링은 method 결합점만 제공
pointcut | advice가 적용되는 대상을 지정. 패키지이름/클래스이름/메서드이름을 정규식으로 지정하여 사용
target | advice가 적용되는 클래스를 의미(aspect를 정의하는 곳)
weaving | advice를 주기능에 적용하는 것을 의미
  
### 스프링에서 AOP기능을 구현  

- 스프링 AOP 사용하기 위해 의존성 추가
  
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```  
- @Aspect 애너테이션을 이용
  
```java
@Component
@Aspect
public class PerfAspect {

@Around("execution(* com.saelobi..*.EventService.*(..))")
public Object logPerf(ProceedingJoinPoint pjp) throws Throwable{
long begin = System.currentTimeMillis();
Object retVal = pjp.proceed(); // 메서드 호출 자체를 감쌈
System.out.println(System.currentTimeMillis() - begin);
return retVal;
}
}
@Around 어노테이션은 타겟 메서드를 감싸서 특정 Advice를 실행한다는 의미이다.
위 코드의 Advice는 타겟 메서드가 실행된 시간을 측정하기 위한 로직을 구현.
추가적으로 execution(* com.saelobi..*.EventService.*(..))가 의미하는 바는 com.saelobi 아래의 패키지 경로의 EventService 객체의 모든 메서드에 이 Aspect를 적용하겠다는 의미

```  
  
경로지정 방식 말고 특정 애너테이션이 붙은 포인트에 해당 Aspect를 실행할 수 있는 기능도 제공함


```java
@Component
@Aspect
public class PerfAspect {

@Around("@annotation(PerLogging)")
public Object logPerf(ProceedingJoinPoint pjp) throws Throwable{
long begin = System.currentTimeMillis();
Object retVal = pjp.proceed(); // 메서드 호출 자체를 감쌈
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
위 출력 결과 @PerLogging 애너테이션이 붙은 메서드만 Aspect가 적용됨.

마찬가지로 스프링 빈의 모든 메서드에 적용할 수 있는 기능도 제공
  
```java
@Component
@Aspect
public class PerfAspect {

@Around("bean(simpleEventService)")
public Object logPerf(ProceedingJoinPoint pjp) throws Throwable{
long begin = System.currentTimeMillis();
Object retVal = pjp.proceed(); // 메서드 호출 자체를 감쌈
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
### Aspect 실행 시점 지정할 수 있는 애너테이션
  
애너테이션 | 기능
---- | ---- 
@Before (이전) | 어드바이스 타겟 메소드가 호출되기 전에 어드바이스 기능을 수행
@After (이후) | 타겟 메소드의 결과에 관계없이(즉 성공, 예외 관계없이) 타겟 메소드가 완료 되면 어드바이스 기능을 수행
@AfterReturning (정상적 반환 이후) | 타겟 메소드가 성공적으로 결과값을 반환 후에 어드바이스 기능을 수행
@AfterThrowing (예외 발생 이후) | 타겟 메소드가 수행 중 예외를 던지게 되면 어드바이스 기능을 수행
@Around (메소드 실행 전후) | 어드바이스가 타겟 메소드를 감싸서 타겟 메소드 호출 전과 후에 어드바이스 기능을 수행
  

### xml파일 AOP 설정  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE beans PUBLIC "-//SPRING//DTD BEAN//EN"
"http://www.springframework.org/dtd/spring-beans-2.0.dtd">
<beans>
  <bean id ="calcTarget" class="com.spring.Calculator">
  <bean id ="logAdvice" class="com.spring.LoggingAdvice">
  <bean id ="proxyCal" class="org.springframework.aop.framework.ProxyFactoryBean">
  -- 스프링에서 제공하는 ProxyFactoryBean을 이용해 타깃과 어드바이스를 엮어줌
    <property name="target" ref="calcTarget"/> -- 타깃 빈을 calcTarget 빈으로 지정
    <property name="interceptorNames">
    -- 스프링의 ProxyFactoryBean의 interceptorNames 속성에 logAdvice를 어드바이스 빈으로 설정하여
    타깃 클래스의 메서드 호출 시 logAdvice를 실행
      <list>
        <value>logAdvice</value>
      </list>
    </property>
  </bean>
</beans>
```  

