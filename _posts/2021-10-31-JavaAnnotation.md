---
title:  "Java Annotation"
excerpt: Java Annotation
categories:
  - Java
---

## 애너테이션이란?
- 자바를 개발한 사람들은 소스코드에 대한 문서를 따로 만들기보다 소스코드와 문서를 하나의 파일로 관리하는 것이 낫다고 생각했다.
- 소스코드의 주석에 소스코드에 대한 정보를 저장하고, 소스코드의 주석으로부터 HTML 문서를 생성해내는 프로그램(javadoc.exe)를 만들어 사용했다.
- 프로그램의 소스코드 안에 다른 프로그램을 위한 정보를 미리 약속된 형식으로 포함시킨 것이 애너테이션
- 애너테이션은 주석과 같이 프로그래밍 언어에 영향을 미치지 않으면서도 다른 프로그램에게 유용한 정보를 제공할 수 있다는 장점이 있다.
-   ```@Test```  애너테이션은 테스트 프로그램인 JUnit에게 테스트를 해야한다고 알리기만 할 뿐 메서드가 포함된 프로그램 자체에는 아무런 영향을 미치지 않는다. 즉, 다른 프로그램에게 어떤 역할 하는지 알려주는 주석이다.
-   ```java.lang.annotation```  에 포함된 JDK 에서 제공하는 표준 애너테이션은 주로 컴파일러를 위한 것으로 컴파일러에게 유용한 정보를 제공한다.
- 애너테이션은 다이나믹하게 실행되는 코드는 들어가지 않는다. 즉, 런타임 중에 알아내야 하는 값은 들어가지 못한다. 컴파일러 수준에서 해석이 되거나, 완전히 정적이어야 하며 동적으로 런타임 중에 바뀌어야 하는 것들은 애너테이션으로 사용할 수 없다.


  
```java
  @RestController
  public DemoController {
    private static final String path = "ex";

    @GetMapping(path)
    public String ex() {
      return "ex";
    }
  }

```  

- path 변수는 static final 정적 변수로 @GetMapping 애너테이션에 사용할 수 있다. 만약 path 변수가 동적으로 할당된다면 컴파일 에러 발생

## 애너테이션 정의

  
```java
  public @interface TestAnnotation {
  }
```  

