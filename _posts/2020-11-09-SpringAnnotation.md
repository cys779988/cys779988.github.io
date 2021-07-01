---
title:  "spring Annotation"
excerpt: spring Annotation 기능
categories:
  - spring
---

### 스프링 애너테이션
- 기존에 XML에서 빈 설정을 애너테이션을 이용해서 자바 코드에서 설정하는 방법
- 기능이 복잡해짐에 따라 XML에서 설정하는 것보다 유지보수에 유리
- 현재 애플리케이션 개발 시 XML 설정 방법과 애너테이션 방법을 혼합해서 사용

### 스프링 애너테이션 제공 클래스
브라우저 URL 요청 처리 애너테이션 관련 클래스
  
클래스 | 기능
---- | ----
DefaultAnnotationHandlerMapping | 클래스 레벨에서 @RequestMapping을 처리
AnnotationMethodHandlerAdapter | 메서드 레벨에서 @RequestMapping을 처리
  
### \<context:component-scan\> 태그 기능
- \<context:component-scan\> 태그를 사용해 패키지 이름을 지정하면 애플리케이션 실행 시 해당 패키지에서 애너테이션으로 지정된 클래스를 빈으로 만들어 줌
- 사용형식
  ```
  <context:component-scan base-package="패키지이름" />
  ```  

### 여러가지 스테레오 타입 애너테이션
  
애너테이션 | 기능
---- | ----
@Controller | 스프링 컨테이너가 component-scan에 의해 지정한 클래스를 컨트롤러 빈으로 자동 변환
@Service | 스프링 컨테이너가 component-scan에 의해 지정한 클래스를 서비스 빈으로 자동 변환
@Repository | 스프링 컨테이너가 component-scan에 의해 지정한 클래스를 DAO 빈으로 자동 변환
@Component | 스프링 컨테이너가 component-scan에 의해 지정한 클래스를 빈으로 자동 변환
  

<img src="https://cys779988.github.io/assets/img/spring-13.png">
<img src="https://cys779988.github.io/assets/img/spring-14.png">

앞에서는 URL요청에 대해 어떤 메소드를 호출할 지 메소드이름-ResolverBean에서 설정했음  
이제 @RequestMapping 애너테이션을 이용해 간단하게 가능

### 애너테이션 이용해 로그인 기능 구현
<img src="https://cys779988.github.io/assets/img/spring-15.png">  
한글 깨짐현상 방지 위해 encodingFilter 추가
<img src="https://cys779988.github.io/assets/img/spring-16.png">  
RequestMethod의 GET방식과 POST방식 둘 다 사용

#### @RequestParam
- 매개변수의 수가 많아지면 일일이 getParameter() 메서드를 이용하는 방법은 불편
- @RequestParam을 메서드에 적용해 쉽게 값을 얻을 수 있음
- getParameter()의 번거로움 없앰

<img src="https://cys779988.github.io/assets/img/spring-17.png">

### RequestParam의 required 속성
- @RequestParam 적용 시 required 속성을 생략하면 기본값은 true
- required 속성을 true로 설정하면 메서드 호출 시 반드시 지정한 이름의 매개변수를 전달해야함
- required 속성을 false로 설정하면 메서드 호출 시 지정한 이름의 매개변수가 전달되면 값을 저장하고 없으면 null할당
- jsp에서 전송될 수도 있고 안될 수도 있는 값은 required 속성 값을 false로 설정
<img src="https://cys779988.github.io/assets/img/spring-18.png">

### @RequestParam 이용해 Map에 매개변수 값 설정하기
- 전송되는 매개변수의 수가 많을 경우 Map에 바로 저장해서 사용하면 편리
<img src="https://cys779988.github.io/assets/img/spring-19.png">
  
- Map<String,String> info
- key값은 전송한 파라미터로 설정, value는 Map의 value로 자동으로 설정하여 info라는 변수로 저장됨

### @ModelAttribute 이용해 VO에 매개변수 값 설정하기
<img src="https://cys779988.github.io/assets/img/spring-20.png">
  
- LoginVO의 동일한 이름을 갖는 속성명에 set함
- @ModelAttribute 사용하면 ModelAndView의 addObject 사용할 필요없이 바로 바인딩 되어 지정한 속성명인 info 호출

### Model 클래스 이용해 값 전달하기
- Model 클래스를 이용하면 메서드 호출 시 JSP로 값을 바로 바인딩하여 전달할 수 있음
- Model 클래스의 addAttribute() 메서드는 ModelAndView의 addObject() 메서드와 같은 기능 수행
- Model 클래스는 따로 뷰 정보를 전달할 필요가 없을 때 사용하면 편리함
<img src="https://cys779988.github.io/assets/img/spring-21.png">

### @Autowired 이용해 빈 주입하기
- XML에서 빈을 설정한 후 애플리케이션이 실행될 때 빈을 주입해서 사용하면 XML 파일이 복잡해지면서 사용 및 관리가 불편함
- 기존 XML 파일에서 각각의 빈을 DI로 주입했던 기능을 코드에서 애너테이션으로 자동으로 수행함
- @Autowired를 사용하면 별도의 setter나 생성자 없이 속성에 빈을 주입할 수 있음
<img src="https://cys779988.github.io/assets/img/spring-22.png">
<img src="https://cys779988.github.io/assets/img/spring-23.png">
<img src="https://cys779988.github.io/assets/img/spring-24.png">
  
- VO 클래스에 @Component 애너테이션 사용
- 모든 클래스에 @Component로 빈 설정 가능하지만 가독성을 위해 서로 다른 애너테이션 사용

### SuppressWarnings
- 컴파일시 컴파일 경고를 사용하지 않도록 설정
  
속성 | 설명
---- | ----
all | 모든 경고를 표시 안함
cast | 캐스트 연산자 관련 경고를 표시 안함
dep-ann | 사용하지 말아야 할 주석 관련 경고를 표시 안함
deprecation | 사용하지 말아야 할 메소드 관련 경고를 표시 안함
fallthrough | switch문에서의 break 누락 관련 경고를 표시 안함
finally | 반환하지 않는 finally 블럭 관련 경고를 표시 안함
null | null 분석 관련 경고를 표시 안함
rawtypes | 제네릭을 사용하는 클래스 매개 변수가 불특정일 때의 경고를 표시 안함
unchecked | 검증되지 않은 연산자 관련 경고를 표시 안함
unused | 사용하지 않는 코드 관련 경고를 표시 안함
  




