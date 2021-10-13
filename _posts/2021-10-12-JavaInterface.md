---
title:  "Java Interface"
excerpt: Java Interface
categories:
  - Java
---

## 인터페이스 정의하는 방법
- 인터페이스란 객체와 객체 사이에서 일어나는 상호작용의 매개로 쓰인다.
- 모든 기능을 추상화로 정의한 상태로 선언
- 인터페이스의 선언은 예약어로 class 대신 interface 키워드를 사용
- 접근제어자로는 public 또는 default 사용
- implements 키워드를 통해 일반 클래스에서 인터페이스를 구현
- Java 8 이전까지는 상수와 추상메서드만 선언가능 (생략하면 위의 타입이 자동으로 붙고, 다른 타입을 사용하면 컴파일 오류가 발생한다.)
- Java 8 이후부터는 default method와 static method가 추가 (두가지 메서드를 통해 구현 강제성 안에 유연함을 추구할 수 있게 되었다.)

  
```java
public interface MyInterface{
	// 상수 : 인터페이스에서 값을 정해주면 바꾸지 못하고 참조만 가능
	String test1 = "Test";
	
	// 추상 메서드 : 가이드만 주고 오버라이딩해서 재구현
	void test2(매개변수, ...);

	// 디폴트 메서드 : 인터페이스에서 기본적으로 제공해주지만, 구현로직을 바꾸고 싶다면 오버라이딩해서 재구현(오버라이딩 하지 않아도 컴파일 오류가 발생하지 않는 메서드)
	default void test3(String param, ...){
		// 구현부
	}

	// 정적 메서드 : 일반적인 static 메서드와 같이 바로 사용가능하며 오버라이딩 할 수 없다.
	static void test4(String param, ...){
		// 구현부
	}
}
```  

- 인터페이스는 추상 클래스와 같이 추상 메서드를 가지므로 추상 클래스와 매우 흡사하다.
- 인터페이스는 추상 클래스와 같이 인스턴스를 생성할 수 없고, 상속받은 클래스에서 구현한 뒤 클래스를 인스턴스화하여 사용한다.

## 추상클래스와 인터페이스 차이점
- 추상 클래스는 일반 메서드와 추상 메서드 둘 다 가질 수 있다.
- 인터페이스는 추상메서드와 상수만을 가진다.(Java8 부터는 default method와 static method를 작성할 수 있다.)
- 인터페이스 내에 존재하는 메서드는 무조건   ```public abstract```  로 선언되며, 이를 생략할 수 있다.
- 인터페이스 내에 존재하는 변수는 무조건   ```public static final```  로 선언되며, 이를 생략할 수 있다.
- 추상클래스는 하나만 상속 가능하지만 인터페이스는 여러개 상속 가능
  
```java
// interface 의 제약 조건을 따르지 않았기 때문에 오류 발생
private int a = 1;

// 컴파일러가 자동적으로 public static final int b = 2로 변경
public int b = 2;

// 컴파일러가 자동적으로 public static final int c = 3으로 변경
static int c = 3;
```  

## 익명 구현 객체
- 소스 파일을 만들지 않고도 구현 객체를 만들 수 있는 방법
- 구현 클래스를 만들어 사용하는 것이 일반적이고, 클래스를 재사용할 수 있기 때문에 편리하지만, 일회성의 구현객체를 만들기 위해 소스파일을 만들고 클래스를 선언하는 것은 비효율적

  
```java
public class App {
	public static void main(String[] args){

		MyInterface myInterface = new MyInterface() {
		    @Override
		    public void turnOn() {

		    }

		    @Override
		    public void turnOff() {

		    }

		    @Override
		    public void setVolume(int volume) {

		    }
		};
	}
}
```  

## 인터페이스 레퍼런스를 통해 구현체를 사용하는 방법

#### 인터페이스를 자료형으로 쓰는 습관을 들이면 프로그램은 훨씬 유용해진다.
- 객체는 인터페이스를 사용해 참조
- 적당한 인터페이스가 있다면 매개변수뿐만 아니라 반환값, 변수, 필드를 전부 인터페이스 타입으로 선언
- 객체의 실제 클래스를 사용할 상황은 오직 생성자로 생성할 때
- 매개변수 타입으로 클래스가 인터페이스를 활용

  
```java
// good. 인터페이스를 타입으로 사용
Set<Son> sonSet = new LinkedHashSet<>();

// bad. 클래스를 타입으로 사용
LinkedHashSet<Son> sonSet = new LinkedHashSet<>();
```  

## 인터페이스 상속
- 인터페이스의 상속 구조에서는 서브 인터페이스는 수퍼 인터페이스의 메서드까지 모두 구현해야 한다.
- 인터페이스 레퍼런스는 인터페이스를 구현한 클래스의 인스턴스를 가리킬 수 있고 해당 인터페이스에 선언된 메서드(슈퍼 인스턴스 메소드 포함)만 호출 할 수 있다.
- 인터페이스는 다중상속이 가능하다.
- 인터페이스간의 상속이 아닌 클래스의 상속에 대해 되짚어 보면, 클래스는 다이아몬드 상속문제로 다중상속이 불가능하다.

  
```java
public class MyClass implements MyInterface1, MyInterface2, MyInterface3 {

}
interface MyInterface1 { }
interface MyInterface2 { }
interface MyInterface3 { }
```  

## Java 8, default Method
- 인터페이스가 default키워드로 선언되면 메소드가 구현될 수 있다. 또한 이를 구현하는 클래스는 default메소드를 오버라이딩 할 수 있다.
- 인터페이스에서 기본적으로 제공해주지만, 구현로직을 바꾸고 싶다면 오버라이딩해서 재구현(오버라이딩 하지 않아도 컴파일 오류가 발생하지 않는 메서드)
- 제약 조건: Object 가 제공하는 기능(equals, hasCode)는 기본 메소드로 제공할 수 없다. (구현체가 재정의 해야함)
- (다이아몬드 문제) 충돌하는 default 메소드의 경우에는 직접 오버라이드 해줘야한다.

  
```java
public class MyClass implements MyInterface {
	// 인터페이스 내부 메소드를 오버라이딩 하지 않았지만 오류가 발생하지 않는다.
	public void doSomething() {
		// 호출해서 사용할 수도 있다.
		test();
	}
}

interface MyInterface {
	default void test() {
		System.out.println("Default Method");
	}
}

```  

## Java 8, static Method
- 해당 인터페이스를 구현한 모든 인스턴스, 해당 타입에 관련되어 있는 유틸리티, 헬퍼 메서드를 제공하고 싶을 때 사용
- 인스턴스없이 수행할 수 있는 작업을 정의할 수 있다.

  
```java
public class MyClass implements {
	// 일반적인 스태틱메서드 사용법과 동일
	MyInterface.test();
}

public interface MyInterface {
	static void test(){
		System.out.println("Static Method");
	}
}
```  

## Java 9, private Method
- 인터페이스에 default method, static method가 생긴 이후, 이러한 메서드들의 로직을 공통화하고, 재사용하기 위해 생긴 메서드
- private method고 default method, static method와 같이 구현부를 가져야하는 제약을 가진다.
- 오직 인터페이스 내부에서만 사용할 수 있다.
- private static 메서드는 다른 static 또는 static이 아닌 메서드에서 사용할 수 있다.
- static이 아닌 private 메서드는 다른 private static 메서드에서 사용할 수 없다.

  
```java
interface CustomCalculator{
	default int addNumbers(int... nums){
		return add(n -> n % 2 != 0, nums);
	}
  
	private int add(IntPredicate predicate, int... nums){
		return IntStream.of(nums)
				.filter(predicate)
				.sum();
	}
}
```  
- 코드의 중복을 피하고 interface에 대한 캡슐화를 유지할 수 있게 되었다.
