---
title:  "Java Lambda"
excerpt: Java Lambda
categories:
  - Java
---

## 람다식(Lambda Expressions)
- JDK 1.8 부터 추가되었으며 메서드를 하나의 식(expression)으로 표현한 것으로 익명함수라고도 한다.
- 람다식은 자바의 새로운 함수 타입 체계는 아니다. 단지 함수형 인터페이스(추상 메서드가 한 개만 존재하는 인터페이스)를 간결한 문법으로 구현할 수 있도록 한 것이다.
- Java 8 람다식은 JVM의 내부 복잡도를 높이는 대신, 기존 명세를 이용하는 방식으로 도입되었다.


## 람다식과 익명 클래스


  
```java

// 익명 클래스
new Object() {
  int max(int a, int b) {
    return a > b ? a : b;
  }
}

// 람다식
(int a, int b) -> a > b ? a : b
```  


위의 익명 객체의 메서드를 호출하기 위해서는 익명 객체의 주소를 참조변수에 저장해야 된다.

  
```java
타입 func = (int a, int b) -> a > b ? a : b;
```  

참조변수는 참조형이므로 클래스 또는 인터페이스가 가능하다. 그리고 람다식과 동등한 메서드가 정의되어 있어야 한다.

  
```java
interface DemoFunction {
    public abstract int max(int a, int b);
}

DemoFunction func = new DemoFunction() {
                public int max(int a, int b) {
                    return a > b ? a : b;
                }
            };

int big = func.max(7, 3);
```  

  
```java
DemoFunction func = (a, b) -> a > b ? a : b; // 익명 객체를 람다식으로 대체
int big = func.max(7, 3);  // 익명 객체 메소드 호출
```  

- 익명 객체를 람다식으로 대체 가능한 이유는 람다식도 실제로는 익명 객체이고, DemoFunction 인터페이스를 구현한 익명 객체의 메서드 max()와 람다식의 매개변수 타입, 개수 그리고 반환값이 일치하기 때문이다.
- 람다식은 익명 클래스에서 변환되었지만 내부 구현은 서로 다르다.


## 함수형 인터페이스
- 람다식을 다루기 위한 인터페이스로 하나의 추상 메서드만 존재해야한다.
- static 메서드와 default 메서드의 개수에는 제약이 없다.
- @FunctionalInterface 애노테이션을 정의하면 컴파일 단계에서 추상메서드가 1개만 선언되었는지 확인할 수 있다.

  
```java
// 함수형 인터페이스
@FunctionalInterface
interface DemoFunction {
  void testMethod();	
  
  static void printName() {
    System.out.println("홍길동");
  }
  default void printAge() {
    System.out.println("28");
  }
}
```  

  
```java
// 익명 내부 클래스 방식 정의
public class Main {
  static void testMethod(DemoFunction f) {
    f.testMethod();
  }
  
  public static void main(String[] args) {
    DemoFunction fn = new DemoFunction() {	
      @Override
      public void testMethod() {
        System.out.println("Hello");
      }
    };
  }
}
```  

  
```java
// 람다 형식 정의
public class Main {
  static void testMethod(DemoFunction f) {
    f.testMethod();
  }
  
  public static void main(String[] args) {
    DemoFunction fn_1 = () -> System.out.println("test");
    testMethod(fn_1);
    
    DemoFunction fn_2 = () -> {
      System.out.println("Test");
      System.out.println("Lambda");
    };
    fn_2.testMethod();
  }
}
```  


#### java.util.function 패키지


함수형 인터페이스 | 메서드
---- | ----
java.lang.Runnable | void run()
Supplier<T> | T get()
Consumer<T> | void accept(T t)
Function<T,R> | R apply(T t)
Predicate<T> | boolean test(T t)
UnaryOperator<T> | T apply(T t)



#### 람다식 타입과 형변환
람다식은 익명 객체이고 익명 객체는 타입이 없다. 따라서 대입 연산자의 양변의 타입을 일치시키기 위해 형변환이 필요하다.

  
```java
DemoFunction f = (DemoFunction)(() -> {});
Object obj = (Object)(()->{});  // Error, 함수형 인터페이스로만 형변환 가능
Object obj = (Object)(DemoFunction)(()->{});
String str = ((Object)(DemoFunction)(()->{})).toString();
```  

## Variable Capture
- 람다식에서 외부 지역변수를 참조하는 행위를 Lambda Capturing 이라고 한다.
- 람다에서 접근가능한 변수는 지역변수, static변수, 인스턴스변수 세 가지 종류가 있다. 그 중 지역변수만 변경이 불가능하고 나머지 변수들은 읽기 및 쓰기가 가능.
- 람다는 지역변수가 존재하는 스택에 직접 접근하지 않고, 지역변수를 자신(람다가 동작하는 쓰레드)의 스택에 복사한다.
- 각각의 쓰레드마다 고유한 스택을 갖고 있어서 지역변수가 존재하는 쓰레드가 사라져도 람다는 복사된 값을 참조하면서 에러가 발생하지 않는다.
- 멀티쓰레드 환경에서는 여러 개의 쓰레드에서 람다식을 사용하면서 Lambda Capturing이 계속 발생하는데 이 때 외부 변수 값의 불변성을 보장하지 못하면서 동기화 문제가 발생한다. 이러한 문제로 지역변수는 final, effectively final 제약 조건을 갖게된다.
- 인스턴스변수나 static변수는 스택 영역이 아닌 힙 영역에 위치하고, 힙 영역은 모든 쓰레드가 공유하고 있는 메모리 영역이기 때문에 값의 쓰기가 가능하다.

#### effectively final
람다식 내부에서 외부 지역변수를 참조하였을 때 지역변수는 재할당을 하지 않아야 하는 것을 의미. 즉, final 변수는 아니지만 마치 final처럼 사용되어지는 변수.

  
```java
public class Main {

  static void testMethod(DemoFunction f) {
    f.testMethod();
  }
  
  public static void main(String[] args) {
    Outer outer = new Outer();
    Outer.Inner inner = outer.new Inner();
    inner.method(100);
  }
}

class Outer {
  int val = 10;  
  class Inner {
    int val = 20;
    void method(int i) {
      int val = 30;
//      i = 10;       // ERROR. 상수의 값은 변경할 수 없다.
      DemoFunction f = () -> {
        System.out.println("i : " + i);
        System.out.println("val : " + val);
        System.out.println("this.val : " + ++this.val);
        System.out.println("Outer.this.val : " + ++Outer.this.val);
      };
      f.testMethod();
    }
  }
}
```  

  
```
i : 100
val : 30
this.val : 21
Outer.this.val : 11
```  

- 람다식 내에서 참조하는 지역변수는 final이 붙지 않아도 상수로 취급되어 값을 변경할 수 없다.
- 반면, Inner 클래스와 Outer 클래스의 인스턴스 변수인 this.val과 Outer.this.val은 상수로 간주되지 않으므로 값을 변경할 수 있다.
- 람다식에서 this는 내부적으로 생성되는 익명 객체의 참조가 아니라 람다식을 실행한 객체의 참조이다. 따라서 위의 코드에서 this는 중첩 객체 Inner를 가리킨다.

## 메서드, 생성자 레퍼런스

#### 메서드 참조
- 람다식이 하나의 메서드만 호출하는 경우에는 메서드 참조를 통해 람다식을 간략하게 작성할 수 있다.

  
```java
// 메서드
Integer wrapper(String s) {
  return Integer.parseInt(s);
}

// 람다식
Function<String, Integer> f = s -> Integer.parseInt(s);

// 메서드 참조
Function<String, Integer> f = Integer::parseInt;

```  

- 메서드 참조에서 람다식 일부가 생략되었지만, 컴파일러는 생략된 부분을 우변의 parseInt 메서드의 선언부로부터 또는 좌변의 Function 인터페이스에 지정된 제네릭 타입으로부터 쉽게 알아낸다.
- 이미 생성된 객체의 메서드를 람다식에서 사용한 경우에는 클래스 이름 대신 그 객체의 참조변수를 적는다.

  
종류 | 람다 | 메서드참조
---- | ---- | ----
static 메서드 참조 |   ```x -> ClassName.method(x)```   | ClassName::method
인스턴스 메서드 참조 |   ```(obj,x) -> obj.method(x)```   | ClassName::method
특정객체 인스턴스 메서드 참조 |   ```x -> obj.method(x)```   | obj::method
  
#### 생성자의 메서드 참조

  
```java

// 람다식
Supplier<DemoClass> s = () -> new DemoClass();
Function<Integer, DemoClass> f = i -> new DemoClass(i);
Function<Integer, int[]> f2 = x -> new int[x];

// 메서드 참조
Supplier<DemoClass> s = DemoClass::new;
Function<Integer, DemoClass> f = DemoClass:new;
Function<Integer, int[]> f2 = int[]::new;
```  


## 람다식의 컴파일
- 람다식은 컴파일 타임에는 객체를 생성하지 않는다. 다만 런타임에 JVM이 객체를 생성할 수 있도록 람다식을 invokedynamic으로 변환하여 '생성하기 위한 정보'를 만들어둔다.
- invokedynamic은 bootstrap method, static args, dynamic args 3가지 정보를 필요로 한다.
- bootstrap method는 동적으로 객체를 생성하는 메서드이다. 람다식에서는 LambdaMetafactory.metafactory()가 bootstrap method에 해당된다.
- static args는 정적인수로 람다식으로 구현한 인터페이스와 람다식 body 코드가 그대로 private static method로 생성되어 전달된다.
- dynamic args는 동적인수로 free variable이 전달된다.

#### free variable
free variable은 자신을 감싼 영역(closure) 외부에 존재하는 변수를 의미한다.

  
```java
int offset = 10; // free variable에 해당
Function<Integer, Integer> func = (a) -> offset * a;
```  


## 람다식의 런타임
- 컴파일 타임에 제시해둔 생성정보를 토대로 런타임에 JVM이 람다 객체를 생성한다. 앞서 언급한 bootstrap method인 LambdaMetafactory.metafactory()가 람다식의 클래스를 동적으로 정의하고 객체를 반환하는 역할을 한다.
- JVM이 동적으로 정의한 람다 클래스는 static args로 넘겨 받은 타겟 인터페이스를 구현한다.
- JVM이 동적으로 정의한 람다 클래스는 실행 메소드(Runnable의 경우 run)에서, 컴파일 타임에 본래 클래스에 생성된 람다식 내부 코드를 담은 private static method을 호출한다.
- 호출된 본래 클래스의 private static method가 실제 람다식 내부 코드를 수행한다.
- 람다 클래스 정의 및 구현에 대한 구체적인 규약은 없다. 동적으로 정의하는 만큼 그 시점에서 효율적인 방식으로 JVM이 유연하게 객체를 생성할 수 있다.

## 람다식의 효율성

#### Stateless 람다(Non Capturing)

  
```java

// 익명 클래스
public void loopAnonymous() {
  myStream.forEach(new Consumer<Item> () {
    @Override
    public void accept(Item item) {
      item.doSomething();
    }
  }
}

// 람다식
public void loopLambda() {
  myStream.forEach(item -> item.doSomething());
}
```  


- 익명클래스로 구현한 forEach문은 매 iteration마다 익명 클래스를 정의하고, 그 객체를 생성한다. 만일 myStream의 요소가 100만개라면, 100만개의 새로운 익명클래스가 생겨난다.
- 람다식으로 구현된 forEach문은 단 한 개의 클래스만이 생겨난다. 람다식으로 생성된 클래스는 그 실행 메서드에서 단순히 본래 클래스의 private static method만 호출하기 때문에 구현 객체만 생성해도 무한하게 재사용할 수 있다.
- 따라서 익명클래스에 비해 성능과 자원 면에서 훨씬 효율적이다.


#### Stateful 람다(Capturing)

  
```java
public class Test {
  public static void main(String[] args) {
    Test test = new Test();
    test.testMethod();
  }
  public void testMethod() {
    int a = 10;
    Runnable runnable = () -> {
      System.out.println("a: " + a);
    };
    runnable.run();
  }
}
```  

  
```java
// -Djdk.internal.lambda.dumpProxyClasses 옵션을 사용해 동적 클래스를 저장
import java.lang.invoke.LambdaForm.Hidden;
// $FF: synthetic class
final class Test$$Lambda$1 implements Runnable {
  private final int arg$1;
  private Test$$Lambda$1(int var1) {
    this.arg$1 = var1;
  }
  private static Runnable get$Lambda(int var0) {
    return new Test$$Lambda$1(var0);
  }
  @Hidden
  public void run() {
    Test.lambda$testMethod$0(this.arg$1);
  }
}
```  

- stateless 람다일 때와 달리 생성자를 통해 free variable의 값을 받아 멤버변수에 대입한다.
- free variable이 local variable이면(stack에 존재) 값이 copy 되고, heap variable이면 reference를 갖는다.
- 람다에서 참조할 수 있는 free variable은 final이거나 effective final 이어야 하므로 그 값이 변경되지 않는다. 따라서 stateful 람다일 때도 마찬가지로 하나의 동적 클래스만으로 계속 재사용이 가능하다.
