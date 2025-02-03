---
title:  "spring boot annotation"
excerpt: spring boot annotation
categories:
  - spring
---
## Spring boot annotation 정리

### @SpringBootApplication
- @Configuration, @EnableAutoConfiguration, @ComponentScan 3가지를 합친 Annotation
- 스프링 부트의 핵심 Annotation으로 메인클래스에 사용

### @Bean
- 개발자가 직접 제어 불가능한 외부 라이브러리 또는 설정을 위한 클래스를 Bean으로 동록할 때 사용

### @Configuration
- 해당 클래스가 설정 파일임을 알려주는 용도
- 자바기반의 설정을 선언할 수 있음
- 1개 이상의 @Bean을 제공하는 클래스의 경우 반드시 @Configuration 명시해줘야함

### @EnableAutoConfiguration
- 스프링 애플리케이션 컨텍스트를 만들 때 자동으로 설정하는 기능을 실행
- classpath의 내용에 기반해서 자동생성.
- 만약 tomcat-embed-core.jar가 존재하면 톰캣 서버가 setting

### @ComponentScan
- 자동으로 컴포넌트 클래스를 검색하고 검색된 컴포넌트 및 빈클래스를 Spring ApplicationContext에 등록하는 역할
- 메인 클래스가 위치한 루트패키지부터 모든 클래스를 검색하여 bean으로 등록

### @RestController
- @Controller와 @ResponseBody 합친 Annotation으로 메소드의 반환결과를 JSON 형태로 반환
- @Controller : api와 view를 동시에 사용하는 경우에 사용. View return 이 주 목적
- @RestController : view가 필요없는 api만 지원하는 서비스에서 사용, context에 설정된 resolver를 무시함

### @GetMapping
- @RequestMapping(Method=RequestMethod.GET) 와 같음
- Http head에 담아 보내기 때문에 용량제한 있음

### @PostMapping
- @RequestMapping(Method=RequestMethod.POST) 와 같음

### @Getter, @Setter
- 클래스 내 모든 필드의 getter, setter 메소드를 자동으로 생성
- Lombok 라이브러리에 의해 getter, setter 메소드를 지원

### @Required
- setter 메소드에 적용시키면 빈 생성시 필수 프로퍼티 임을 알림

### @Qualifier("test")
- @Autowired와 같이 쓰이며, 같은 타입의 빈 객체가 있을 때 해당 아이디를 적어 원하는 빈이 주입될 수 있도록 함

### @Resource
- @Autowired와 마찬가지로 빈 객체를 주입해주는데 차이점은 @Autowired는 타입으로, @Resource는 이름으로 연결

### @Scope
- 스프링은 기본적으로 빈의 범위를 singleton으로 설정. singleton이 아닌 다른 범위를 지정하고 싶다면 @Scope 이용하여 범위 지정

### @PostConstruct, @PreConstruct
- 의존하는 객체를 생성한 이후 초기화 작업을 위해 사용
- 객체 생성 전/후(pre/post)에 실행해야 할 메소드 앞에 붙임
- 사용하려면 CommonAnnotationBeanPostProcessor 클래스를 빈으로 등록하거나 <context:annotation-config> 태그 추가

### @PreDestroy
- 컨테이너에서 객체를 제거하기 전(pre)에 해야할 작업을 수행하기 위해 사용
- 사용하려면 CommonAnnotationBeanPostProcessor 클래스를 빈으로 등록하거나 <context:annotation-config> 태그 추가

### @PropertySource
- 해당 프로퍼티 파일을 Environment로 로딩하게 해줌.
- 클래스에 @PropertySource("classpath:/settings.properties")라고 적고 클래스 내부에 @Resource를 Environment타입의 멤버변수 앞에 적으면 매핑 됨

### @SessionAttributes
- 세션상에서 model의 정보를 유지하고 싶을 경우 사용
-   ```@SessionAttributes("blog")```   model로 할당된 객체 중 지정된 이름과 일치하는 객체를 세션 속성에도 추가함

### @PathVariable
- URI의 일부를 파라미터 혹은 변수로 사용

### @ConfigurationProperties
- yaml파일을 읽음. default로 classpath:application.properties 파일이 조회됨

### @Lazy
- 지연로딩을 지원
- @Component나 @Bean Annotation과 같이 쓰이며 클래스가 로드될 때 스프링에서 바로 bean등록을 마치는 것이 아니라 실제로 사용될 때 로딩이 이뤄지게 하는 방법

### @Value
- properties에서 값을 가져와 적용할 때 사용
-   ```@Value("${value.file}")```  spEL을 사용가능


### @Valid
- 유효성 검증이 필요한 객체임을 지정

### @InitBinder
- @Valid 로 유효성 검증이 필요하다고 한 객체를 가져오기 전에 수행해야할 메소드를 지정

### @RequestAttribute
- Request에 설정되어 있는 속성 값을 가져옴

### @RequestBody
- 요청이 온 데이터(JSON이나 XML 형식)를 바로 클래스나 model에 매핑

### @RequestPart
- Request로 온 MultipartFile을 바인딩
-   ```@RequestPart("file")MultiPartFile file```  

### @CookieValue
- 쿠키 값을 파라미터로 전달 받을 수 있는 방법
- required 속성이 true일 때 해당 쿠키가 존재하지 않으면 500 error, false일 때는 에러를 발생시키지 않음

### @Transactional
- 데이터베이스 트랜잭션을 설정하고 싶은 메소드에 적용하면 메소드 내부에서 일어나는 데이터베이스 로직이 모두 성공하거나 하나라도 실패하면 롤백

### @Cacheable
- 메소드 앞에 지정하면 해당 메소드를 최초에 호출할 때 캐시에 적재하고 다음부터는 동일한 메소드 호출이 있을 때 캐시에서 결과를 가져와서 리턴하므로 메소드 호출 횟수를 줄임
- 메소드에 호출에 사용되는 자원이 많고 자주 변경되지 않을 때 사용하고 나중에 수정되면 캐시를 없애는 방법을 활용

### @CachePut
- 캐시를 업데이트하기 위해 메소드를 항상 실행하게 강제하는 Annotation
- 해당 Annotation이 있으면 항상 메소드호출을 하게되므로 @Cacheable과 같이 사용하면 안됨

### @CacheEvict
- 캐시에서 데이터를 제거하는 트리거로 동작하는 메소드

### @CacheConfig
- 클래스 레벨에서 공통의 캐시설정을 공유하는 기능

### @Scheduled
- 정해진 시간에 실행해야할 경우 사용
-   ```@Scheduled(cron="0 0 01 * * ?"```  초 분 시 일 월 요일 년(선택)에 해당 메소드 호출

### @CrossOrigin
- CORS 보안상의 문제로 브라우저에서 리소스를 현재 origin에서 다른 곳으로의 ajax 요청을 방지
- @RequestMapping이 있는 곳에 사용하면 해당 요청은 타 도메인에서 온 ajax요청을 처리
-   ```@CrossOrigin(origins="http://test.com", maxAge = 3600)```   기본 도메인이 http://test.com 인 곳에서 온 ajax요청만 받아줌

