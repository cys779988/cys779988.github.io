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

- 애너테이션은 element라는 것을 멤버로 가질 수 있다.

  
```java
  @TestAnnotation(name = "chae", age = 19)
  public class Test {
  }
  
  @interface TestAnnotation {
    String name();
    int age();
  }
```  

- 애너테이션에 기본값 설정

  
```java
  @TestAnnotation
  public class Test {
  }
  
  @interface TestAnnotation {
    String name() default "chae";
    int age() default 19;
  }
```  

- 애너테이션은 value라는 기본 element를 가질 수 있다.
- value라는 element는 애너테이션을 적용할 때 굳이 element의 이름을 명시해주지 않아도 된다.
- 두 개 이상의 element 값을 할당해야하는 경우 value 명시
  
```java
  @TestAnnotation("chae")
  public class Test {
  }
  
  @interface TestAnnotation {
    String value();
  }
```  

## 자바 표준 애너테이션
- 자바에서 기본적으로 제공하는 애너테이션으로 built-in annotation 이라고도 한다.

애너테이션 | 설명
---- | ----
@Override | 오버라이딩 된 메서드에 사용
@Deprecated | 사용을 권장하지 않는 메서드에 사용
@SuppressWarnings | 컴파일 경고를 무시하도록 할 때 사용
@SafeVarargs | 가변인자 parameter 사용시 경고를 무시할 때 사용(JDK 1.7이상)
@FunctionalInterface | 함수형 인터페이스임을 명시할 때 사용(JDK 1.8이상)

## 자바 메타 애너테이션
- 애너테이션을 위한 애너테이션
- 애너테이션을 정의할 때 애너테이션의 적용대상, 유지기간 등을 지정하는데 사용한다.

#### @Target
- 애너테이션 적용가능 대상을 지정하는데 사용
- 애너테이션과 관련하여 java.lang.annotation.ElementType 이라는 enum 타입이 정의되어 있다.


ElementType | 적용대상
---- | ----
TYPE | class, interface, enum
ANNOTATION_TYPE | annotation
FIELD | field
CONSTRUCTOR | constructor
METHOD | method
LOCAL_VARIABLE | local variable
PACKAGE | package

  
```
@Target({ElementType.FIELD, ElementType.METHOD})
@interface TestAnnotation {
}

```  

#### @Retention
- 애너테이션 정보를 언제까지 유지할 것인지 정하는데 사용
- 이 유지 정책은 java.lang.annotation.RetentionPolicy 라는 enum 타입으로 정의되어 있다.


유지정책 | 설명
---- | ----
SOURCE | 소스 파일에만 존재. 클래스 파일(바이트코드)에는 존재하지 않음
CLASS(기본값) | 클래스 파일에 존재. 실행 시에는 사용불가(리플렉션으로 정보를 얻을 수 없음)
RUNTIME | 클래스 파일에 존재. 실행 시에 사용가능(리플렉션으로 정보를 얻을 수 있음)

  
```
public class Test {
  public static void main(String[] args) throws ClassNotFoundException {
    Class clazz = Class.forName("com.spring.ReflectionTestClass");
    Annotation[] annotations = clazz.getDeclaredAnnotations();
    for (Annotation annotation : annotations) {
      System.out.println("annotation : " + annotation.toString());
    }
  }
}

@Retention(RetentionPolicy.RUNTIME)
@interface TestAnnotation { }

@MyAnnotation_04
class ReflectionTestClass { }

```  

- @Retention 으로 기본으로 설정된 CLASS 유지정책을 RUNTIME으로 변경하면 리플렉션으로 클래스의 정보를 얻을 수 있다.


#### @Documented
- 애너테이션에 대한 정보가 javadoc으로 작성한 문서에 포함되도록 한다.
- 자바에서 제공하는 기본 애너테이션 중 @Override, @SuppressWarnings를 제외한 모든 표준 애너테이션에 붙어있다.

