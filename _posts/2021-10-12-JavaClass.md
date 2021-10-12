---
title:  "Java 클래스와 상속"
excerpt: Java 클래스, 상속
categories:
  - Java
---

## Class란?
- OOP에서 특정 객체를 생성하기 위해 변수와 메서드를 정의하는 일종의 틀
- 클래스는 그 자체로는 사용할 수 없고 객체화 되어야 실제로 사용할 수 있다.
- java 파일 안에 다수의 클래스를 정의할 수 있지만 public 키워드가 붙은 class는 단 하나만 존재해야 하며 이 클래스는 파일이름과 동일한 이름을 가져야 한다.

## new 키워드
- new 키워드를 사용하면 객체를 동적 메모리 할당 영역(Heap 영역)에 저장할 공간을 할당하고 저장한 공간의 주소를 객체에 반환한다.(GC에 의해 관리된다는 것을 의미)
- 클래스로부터 객체를 만드는 과정을 클래스의 인스턴스화라고 한다.
- 클래스로부터 만들어진 객체를 클래스의 인스턴스라고 한다.

  
```
public static void main(String[] args) {
  MyClass test1 = new MyClass();  // test1: MyClass@798
  MyClass test2 = new MyClass();  // test2: MyClass@799
}
```  

- new 연산자를 통해 MyClass라는 객체를 2개 생성했을 때 test1과 test2의 주소값이 다르다.

## 자바의 상속
- 상속을 해주는 클래스가 가진 멤버 필드와 멤버 메서드를 상속을 받는 클래스에서 마치 자신의 것처럼 사용할 수 있는 것
- 다중상속을 지원하지 않음
- 계층구조의 최상위에 있는 클래스는 Object클래스
- super 키워드를 이용하여 부모의 필드에 접근

## 메서드 오버라이딩
- 슈퍼클래스와 서브클래스의 메서드에서 발생하는 관계
- 슈퍼클래스의 메서드를 동일한 이름으로 서브클래스에서 재작성하는 행위

  
```
public Class Parent {
  void print() {
    System.out.println("Parent");
  }
}

public Class Child extends Parent {
  void print() {
    System.out.println("Child");
  }
}

```  

- 동적바인딩을 통해 오버라이딩 된 메서드가 항상 우선적으로 호출된다.
- 부모 클래스의 메서드가 public이면 오버라이딩 하는 메서드의 접근 지정자는 public만 사용가능(protected면 protected, public 가능)
- static, private, final로 선언된 메서드는 오버라이딩 불가

## 추상클래스(abstract class)
- 하나 이상의 추상 메서드를 포함하거나 abstract로 선언한 클래스
- 추상클래스는 인스턴스를 생성할 수 없다.
- 추상클래스를 상속받는 자식클래스는 반드시 추상 메서드를 재정의(구현) 해야한다.

## 다이나믹 메서드 디스패치(Dynamic Method Dispatch)
- 정적 디스패치 (Static Dispatch)  :  컴파일 시점에 호출 할 메서드가 결정되는 경우
- 동적 디스패치 (Dynamic Dispatch) : 컴파일 시점에서는 어떤 메서드가 실행될지는 모르고 런타임에 어떤 메서드가 실행되는지 결정되는 경우

## final
- 자바에서 상수(constant)로 다른 값을 가질 수 없다.
- final 키워드를 사용하면 초기에 값을 할당해야하고 다른 값을 가질 수 없다.
- final 메서드의 경우 오버라이드 할 수 없다.
- final 클래스의 경우 상속할 수 없다. 즉, 부모클래스가 될 수 없음을 뜻한다. (부모클래스를 가질 수는 있다.)

## Object
- 모든 클래스의 최상위 부모클래스

  
메서드 | 설명
---- | ----
boolean equals(Object obj) | 두 개의 객체가 같은지 비교하여 같으면 true를, 같지 않으면 false를 반환한다.
String toString() | 현재 객체의 문자열을 반환한다.
protected Object clone() | 객체를 복사한다.
protected void finalize() | 가비지 컬렉션 직전에 객체의 리소스를 정리할 때 호출한다.
Class getClass() | 객체의 클래스형을 반환한다.
int hashCode() | 객체의 코드값을 반환한다.
void notify() | wait된 스레드 실행을 재개할 때 호출한다.
void notifyAll() | wait된 모든 스레드 실행을 재개할 때 호출한다.
void wait() | 스레드를 일시적으로 중지할 때 호출한다.
void wait(long timeout) | 주어진 시간만큼 스레드를 일시적으로 중지할 때 호출한다.
void wait(long timeout, int nanos) | 주어진 시간만큼 스레드를 일시적으로 중지할 때 호출한다.
boolean equals(Object obj) | 두 개의 객체가 같은지 비교하여 같으면 true를, 같지 않으면 false를 반환한다.
