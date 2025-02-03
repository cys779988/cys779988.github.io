---
title:  "Java Exception"
excerpt: Java Exception
categories:
  - Java
---

## Error와 Exception
- Error와 Exception은 모두 Throwable 클래스의 하위 클래스인데, Error는 보통 시스템 리소스 문제로 발생하게 되는데 보통 시스템 충돌, 메모리 부족이 있고 Exception은 컴파일 시간과 런타임시 발생하는게 대부분

<img src="https://cys779988.github.io/assets/img/java(6).png">

#### Error
- 컴퓨터 하드웨어의 오동작 또는 고장으로 인해 응용프로그램에 이상이 생겼거나 JVM 실행에 문제가 생겼을 경우 발생
- 컴파일단계에 발생할 수 없다.
- 프로세스에 영향을 준다.
- 시스템 레벨에서 발생(자바 프로그램 외의 오류)
- VirtualMachineError, OutOfMemoryError, StackOverflowError

#### Exception
- 컴퓨터의 에러가 아닌 사용자의 잘못된 조작 또는 개발자의 잘못된 코딩으로 인해 발생하는 프로그램 오류
- 런타임, 컴파일단계에 발생
- 스레드에 영향을 준다.
- 예외가 발생하면 프로그램이 종료가 된다는 것은 에러와 동일하지만 예외는 예외처리(Exception Handling)을 통해 프로그램을 종료시키지 않고 정상적으로 작동되게 만들어줄 수 있다.
- 자바에서 예외처리는 try-catch문을 통해 사용한다.

#### RuntimeException
- 예외처리를 통해 처리할 수 있는 예외
- 모든 예외 클래스는 java.lang.Exception에 저장되어 있다.
- 일반예외(Exception) : 일반예외와 실행예외 클래스를 구별하는 방법은 일반예외는 Exception을 상속받지만, RuntimeException은 상속받지 않음
- 실행예외(RuntimeException) : 실행예외는 java.lang.Exception 하위 클래스 java.lang.RuntimeException을 상속, JVM에서는 RuntimeException 상속 여부를 보고 판단

## CheckedException과 UncheckedException

#### CheckedException
- 체크가 가능한 예외
- 체크란 compiler 단에서 체크 가능한 에러, RuntimeException과 상반된다고 할 수 있다.
- 체크된 에러들, 즉 에러가 발생할 수 있을 때에는 예외처리를 진행해줘야 하는데 자바에서 강제적으로 try-catch문을 작성하도록 하게된다.

  
```
	try {
		Class clazz = Class.forName("java.lang.Strang2");
	} catch (ClassNotFoundException e) {
		// TODO: handle exception
		e.printStackTrace();
	}
```  

- Class.forName(String str)에서 str 매개변수의 클래스가 존재하면 Class 객체를 리턴하지만 존재하지 않으면 ClassNotFoundException 예외를 발생시키게 된다.
- ClassNotFoundException 예외는 일반 예외이므로 컴파일러는 개발자가 예외처리를 작성하도록 만들게된다.

#### UncheckedException
- 컴파일러 자체에서 찾아내지 못하는 예외
- CheckedException과 다르게 예외처리를 강제로 하지않는다.

  
```
	int arr[] = {1, 2, 3, 4, 5};
	System.out.println(arr[6]);
	// java.lang.ArrayIndexOutOfBoundsException 발생
```  

- try-catch를 사용해야할지 컴파일러가 명시해주지 않기 때문에, 개발자의 경험 또는 테스트를 통해 예외처리를 해줄 수 있도록 해야한다.

## 예외처리방법

  
```
	try {

	  // 예외발생코드
	} catch (e1) {
	  // e1 예외 발생 시 처리코드
	} catch (e2) {
	  // e2 예외 발생 시 처리코드
	} finally {
	  // 항상 실행되는 코드
	}
```  


## Java 7, Multi catch
- Java 7 이상부터 여러개의 catch문들을 효과적으로 다룰 수 있도록 제공
- 복잡한 여러개의 catch scope를 한번에 한개의 스코프로 합쳐주게 된다.

  
```
	try {
	// 예외발생코드
	} catch (예외1 | 예외2 e) {
	  // e1 예외 발생 시 처리코드
	} finally {
	  // 항상 실행되는 코드
	}
```  

  
```
	try {
		Class clazz = Class.forName("java.lang.Strang2");
	} catch (ClassNotFoundException | NullPointerException e) {
		e.printStackTrace();
	} finally {

	}
```  

## Java 7, try-catch-resource
- Java 7 이상부터 InputStream, 또는 네트워크프로그래밍에서 사용하는 Socket, ServerSocket을 좀 더 안전하게 사용할 수 있도록 제공
- return 하기전 close() 메서드를 실행해줘야하는 불편함 해소

  
```
	FileInputStream fis = null;
	try {
		fis = new FileInputStream("MyFile.txt");
	} catch (IOException e) {
		e.printStackTrace();
	} finally {
		if (fis != null) {
			try {
				fis.close();
			} catch (IOException e) {
				e.printStackTrace();
				// 또 다른 try-catch 문을 작성해줘야함
			}
		}
	}
```  


  
```
	try (FileInputStream fis = new FileInputStream("MyFile.txt")) {
		// try() Exception 발생 시 fis.close() 메소드 실행
		// 명시적으로 fis.close() 를 적어주지 않는다.
		System.out.println(fis.read());
	} catch (IOException e) {
		e.printStackTrace();
	}
```  

## 예외 던지기 throws
- 기본적으로 예외처리는 try-catch문으로 하는게 보통이지만 자신을 호출한 메서드에 Exception문을 넘겨 처리할 수 있다.

  
```
	public static void method1() {
		try {
			method2();
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		} finally {
			System.out.println("Finish");
		}
	}
	
	private static void method2() throws ClassNotFoundException {
		Class clazz = Class.forName("java.lang.String2");		
	}
	
	public static void main(String[] args) {
		method1();
	}
```  
- method1에서 method2를 처리하는 도중 예외가 발생하면, method1에 예외를 던질 수 있다. 이를 예외 떠넘기기라고 한다.

## 사용자 정의 예외

  
```
	public class CustomException extends Exception{
		public CustomException() {
		}
		public CustomException(String message) {
			super(message);
		}
	}
```  

  
```
	public void method() throws CustomException {
		throw new CustomException("예외처리");
	}
```  
