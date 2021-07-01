---
title:  "spring Constructor Injection"
excerpt: spring 생성자 주입
categories:
  - spring
---

## 의존성 주입 방법
### 생성자 주입(Constructor Injection)
  
```java
@Service 
public class UserServiceImpl implements UserService {
  private UserRepository userRepository;
  private MemberService memberService;

  @Autowired
  public UserServiceImpl(UserRepository userRepository, MemberService memberService) {
    this.userRepository = userRepository;
    this.memberService = memberService; 
  }
}
```  

- 생성자 주입은 생성자의 호출 시점에 1회 호출 되는 것이 보장됨. 그래서 주입받은 객체가 변하지 않거나 반드시 객체의 주입이 필요한 경우에 강제하기 위해 사용함
- 스프링에서는 생성자가 1개만 있을 경우에 @Autowired를 생략해도 주입이 가능하도록 편의성 제공
- 위의 코드는 아래와 같음
  
```java
@Service
public class UserServiceImpl implements UserService {
  private UserRepository userRepository;
  private MemberService memberService;

  public UserServiceImpl(UserRepository userRepository, MemberService memberService) {
    this.userRepository = userRepository;
    this.memberService = memberService;
  }
}
```  

### 수정자 주입(Setter Injection)
- 필드 값을 변경하는 Setter를 통한 의존 관계 주입 방법
- 생성자 주입과 다르게 주입받는 객체가 변경될 가능성이 있는 경우에 사용
  
```java
@Service 
public class UserServiceImpl implements UserService {
  private UserRepository userRepository;
  private MemberService memberService;

  @Autowired
  public void setUserRepository(UserRepository userRepository) {
    this.userRepository = userRepository;
  }
  
  @Autowired
  public void setMemberService(MemberService memberService) {
    this.memberService = memberService;
  }
}
```  

### 필드 주입(Field Injection)

  
```java
@Service
public class UserServiceImpl implements UserService {
  @Autowired
  private UserRepository userRepository;
  @Autowired
  private MemberService memberService;
}
```  

- 필드 주입을 이용하면 코드가 간결해져서 과거에 상당히 많이 이용되었던 주입 방법
- 필드 주입은 외부에서 변경이 불가능하다는 단점이 존재함
- 테스트 코드의 중요성이 부각됨에 따라 필드의 객체를 수정할 수 없는 필드 주입은 거의 사용되지 않게 됨
- 필드 주입은 반드시 DI 프레임워크가 존재해야 하므로 사용을 지양함

### 일반 메소드 주입(Method Injection)
- 거의 사용하지 않는 방법

## 생성자 주입을 사용해야 하는 이유

1. 객체의 불변성 확보  
의존 관계 주입의 변경이 필요한 상황은 거의 없음 하지만 수정자 주입이나 일반 메소드 주입을 이용하면 불필요하게 수정의 가능성을 열어두게 되며, 이는 OOP의 5가지 개발 원칙 중 OCP(개방-폐쇄의 원칭)
를 위반하게 됨. 그러므로 생성자 주입을 통해 변경의 가능성을 배제하고 불변성을 보장

2. 테소트 코드의 작성  
필드 주입으로 작성된 경우 순수 자바코드로 단위 테스트 작성이 불가능 반면 생성자 주입을 사용하면 컴파일 시점에 객체를 주입받아 테스트 코드를 작성하므로 주입하는 객체가 누락된 경우 컴파일 시점에 오류를 발견할 수 있음

3. final 키워드 작성 및 Lombok과의 결합  
생성자 주입을 사용하면 필드 객체에 final 키워드를 사용할 수 있으며, 컴파일 시점에 누락된 의존성을 확인할 수 있음. 반면에 생성자 주입을 제외한 다른 주입 방법들은 객체의 생성(생성자 호출) 이후에 호출되므로 final 키워를 사용할 수 없음
또한 final 키워드를 붙임으로써 Lombok과 결합되어 코드를 간결하게 작성. Lombok에는 final 변수를 위한 생성자를 대신 생성해주는 @RequiredArgsConstructor 제공
  
```java
@Service 
@RequiredArgsConstructor
public class UserServiceImpl implements UserService {
  private final UserRepository userRepository;
  private final MemberService memberService;
  
  @Override
  public void register(String name) {
    userRepository.add(name);
  }
}
```  

- 이러한 코드가 가능한 이유는 Spring에서는 생성자가 1개인 경우 @Autowired를 생략할 수 있도록 하며, 해당 생성자를 Lombok으로 구현했기 때문

4. 순환 참조 에러 방지  
애플리케이션 구동시점(객체의 생성 시점)에 순환 참조 에러를 방지. Bean에 등록하기 위해 객체를 생성하는 과정에서 다음과 같은 순환 참조가 발생하기 때문
  
```java
new UserServiceImpl(new MemberServiceImpl(new UserServiceImpl(new MemberServiceImpl()...)))
```  
