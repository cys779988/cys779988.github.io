---
title:  "Java 데이터 타입, 변수, 배열"
excerpt: Java 데이터 타입, 변수, 배열
categories:
  - Java
---


## Primitive Type(원시타입) 종류, 값의 범위, 기본값

<img src="https://cys779988.github.io/assets/img/java(4).png">

## Primitive Type(원시타입), Reference Type(참조타입)

- Reference Type은 Array, Enumeration, Class, Interface가 있으며 할당메모리는 4 bytes(객체의 주소값), 기본값은 Null 이다.
- Reference Type의 특성은 주소를 저장하는 포인터의 개념으로 동작, 기본값은 Null 이므로 아무런 값을 할당하지 않았을 때 NullPointException이 발생할 수 있기 때문에 Null 체크가 매우 중요하다.
- Primitive Type은 Stack이라고 불리는 메모리 영역에 값과 같이 저장
- Reference Type은 값이 가비지 컬렉션 힙 영역에 저장되고 런타임 스택 영역에 있는 부분은 해당 데이터의 주소값을 가지고 참조하는 형태로 저장되는 타입

## Literal
- 실제로 저장되는 값 그 자체로 메모리에 저장되어있는 변하지 않는 값 그 자체를 뜻한다. 또는 컴파일 타임에 프로그램 안에 정의되어 그 자체로 해석되어야 하는 값을 뜻한다.
- 종류로는 int, double, char, String, boolean 등이 있다.
  
```java
// 정수 리터럴(7이라는 값 자체가 리터럴)
int a = 7;

// 실수 리터럴(0.1234 값 자체가 리터럴)
double f = 0.1234;

// 문자열 리터럴(hello 값 자체가 리터럴)
String val = "hello";
```  

## 변수 선언 및 초기화 방법

#### 변수 선언
  
```
int i;
double f;
char c;
```  

#### 초기화 방법

1. 명시적 초기화

  
```java
class Car {
  int oil = 10; // 원시타입 변수의 초기화
  Engine e = new Engine(); // 참조형 변수의 초기화
  ...
```  

2. 생성자

  
```java
class Car {
  int oil;
  Engine e = new Engine();
  
  Car(int oil) {
    this.oil = oil;
  }
}

Car car = new Car(10); // 10
```  

3. 초기화 블럭
- 조건문, 반복문, 배열, 예외처리 등의 명령문을 통해 초기화 작업이 복잡할 때 사용
- 생성자보다 먼저 수행됨
  
```java
public class Square {
  int x;
  int y;
  
  { // 초기화 블럭
    x = Math.abs(-10);
    x += 10;
    y = 7;
  }
}

Square s = new Square();
```  

## 변수의 스코프와 
- 변수의 스코프는 그 변수에 접근할 수 있는 범위, 자바 언어는 블록스코프( {} 중괄호 )를 사용한다.
- 프로그램 상에서 사용되는 변수들은 사용가능한 범위가 존재한다. 해당 범위가 끝나게 되면 메모리에서 해당 변수가 제거되는 것이 변수의 LifeCycle이다.
- Reference Type의 변수의 라이프 타임은 GC(Garbage Collector)와 관련이 있는데 GC는 가비지 컬렉션 힙 영역에 존재하는 참조 타입 변수의 객체에 대해 동작한다.
- 힙 영역에 메모리가 부족할 경우 GC가 이 영역을 스캔하고, 사용하지 않는 객체를 제거한다.

  
```java
public class GcTest {
  MyTest test = new MyTest();
  test = null;
}
```  

- MyTest 클래스의 객체를 생성해서 test 변수에 할당하면 런타임 스택 영역에 test 변수가 생성되고, 그 값은 가비지 컬렉션 힙 영역에 생성된 new MyTest()로 만들어진 객체가 저장된 주소값을 가진다.
- 이때 런타임 스택 영역의 test 변수의 값인 주소값에 null을 할당하면, new MyTest()로 만든 이 객체는 더이상 아무도 참조하지 않게 된다. (GC의 대상이 됨)
- 마지막으로 런타임 스택 영역에 생성된 변수의 라이프 타임은 블록 스코프에 의존적이기 때문에 변수는 블록이 종료될 때 런타임 스택 영역에서 소멸된다.

## 타입 변환, 캐스팅 그리고 타입 프로모션
- 특정 데이터 타입으로 표현된 리터럴은 다른 데이터 타입으로 변환할 수 있다.
- 변환될 때 두가지 경우
  - 타입 프로모션 : 자신의 표현 범위를 모두 포함한 데이터 타입으로 변환
  - 타입 캐스팅 : 자신의 표현 범위를 모두 포함하지 못한 데이터 타입으로 변환
- float 데이터 타입의 값을 long 타입으로 변환한다면 메모리 크기가 아닌 데이터 표현 범위로 따지기 때문에 정수를 실수로 변환할 때 원본 데이터 손실이 발생하는데, 이것을 타입 캐스팅이라고 한다. 반대로 모두 수용할 수 있다면 타입 프로모션이다.

## 배열 선언
- 배열은 동일한 자료형을 정해진 수만큼 저장하는 순서를 가진 레퍼런스 타입 자료형

  
```java
int[] type_1;
int type_2[];

//값 할당
int[] type_3 = new int[5];
int[] type_4 = {1, 2, 3, 4, 5};   // 변수 선언과 동시에 할당한 경우에만 사용
int[] type_5 = new int[]{1, 2, 3, 4, 5};
```  

- 선언한 배열 변수는 JVM의 런타임 스택 영역에 생성
- 배열은 레퍼런스 타입이기 때문에 값은 가비지 컬렉션 힙 영역에 객체가 생성
- 이 힙 영역의 주소 값이 런타임 스택 영역에 생성된 변수의 값으로 할당

## 타입 추론, var

#### 타입 추론
타입추론(Type inference) 이란 값을 보고 컴파일러가 데이터 타입이 무엇인지 추론한다는 것을 의미
  
```
HashMap<String, Object> myMap = new HashMap<>();
```  

- myMap에서 HashMap 객체를 할당할 때 new HahsMap<String, Object>()를 사용하지 않고, new HashMap<>()(다이아몬드 연산자)를 사용했다.
- 이것은 myMap 변수에 담길 데이터 타입이 HashMap<String, Object> 라는 것을 myMap 변수의 데이터 타입을 바탕으로 추론해낼 수 있기 때문이다.

#### var
- Java 10부터 생겨난 문법
- 로컬 변수이면서 선언과 동시에 값이 할당되어야 함
- var을 실제 타입으로 치환하는 것은 컴파일 타임
