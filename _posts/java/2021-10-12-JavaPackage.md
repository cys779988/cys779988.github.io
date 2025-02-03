---
title:  "Java 패키지"
excerpt: Java 패키지
categories:
  - Java
---


## package 키워드
- 패키지는 관련 클래스와 인터페이스 집합을 구성하는 네임스페이스. 컴퓨터의 디렉토리와 유사하다.
- 자바는 패키지의 가장 상위 디렉토리(root)에서 실행해야한다는 약속이 있기 때문에 해당 패키지로 가서 컴파일 하지 않는다.
- 소스 가장 첫 줄에 있어야하고, 패키지 선언은 소스 하나에 하나만 있어야한다.
- 패키지 이름과 위치한 폴더 이름이 같고 패키지 이름은 java로 시작하면 안된다.
- 모든 클래스에는 정의된 클래스 이름과 패키지 이름이 있다. 이 둘을 합쳐야 완전하게 한 클래스를 표현한다고 할 수 있으며 FQCN(Fully Qualified Class Name)라고 한다.

## import 키워드
- 다른 패키지명에 있는 클래스를 찾지 못할 때 사용
- 보통 IDE의 단축키를 사용함
- 빌트-인 패키지는 import하지 않아도 사용가능하다.

#### 빌트-인 패키지(Built-in Package)
- 개발자들이 사용할 수 있도록 여러 많은 패키지 및 클래스를 제공
- 자바에서 java.lang, java.util 패키지는 기본적인 것들이기 때문에 import 하지 않아도 자바가 알아서 java.lang 클래스를 불러온다.

## 접근 제어자(Access Modifier)

이름 | 설명
---- | ----
public | 누구나 접근 가능
protected | 같은 패키지에 있거나, 상속 받는 경우 사용할 수 있다.
package-private | 아무 접근제어자를 적어주지 않은 경우이며, package-private라 불린다. 같은 패키지 내에서 접근 가능하다.
private | 해당 클래스 내에서만 접근 가능하다.
  

/ | 해당 클래스 내	| 같은 패키지 내 | 상속받은 클래스 | import 한 클래스
---- | ---- | ---- | ---- | ----
public | O | O | O | O
protected | O | O | O | X
package-private | O | O | X | X
private | O | X | X | X
  

## 클래스패스
- 클래스를 찾기 위한 경로
- java에서 설정하는 환경변수는 PATH, CLASSPATH 두가지가 있다.
- PATH는 실행프로그램의 위치   ```[자바설치경로]/bin```  
- CLASSPATH는 실행프로그램에서 사용하는 라이브러리의 위치   ```[자바설치경로]/lib```  
- JVM이 프로그램을 실행할 때 클래스파일을 찾는데 CLASSPATH를 사용한다.
- JVM은 CLASSPATH의 경로를 확인하여 라이브러리 클래스들의 위치를 참조하게 된다. J2JDK 버전부터는 \/jre\/lib\/ext 폴더에 필요한 클래스 라이브러리들을 복사해 놓으면 사용가능하여 특별한 경우가 아니면 설정하지 않는다.
- classpath는 .class 파일이 포함된 디렉토리와 파일을 콜론(;)으로 구분한 목록
- classpath를 지정하기 위한 두가지 방법
  - CLASSPATH 환경변수 사용
  - java runtime에 -classpath 옵션사용

#### CLASSPATH 환경변수 사용
- 컴퓨터 시스템 환경변수 설정을 통해 지정할 수 있다.
- JVM이 시작될 때 JVM의 클래스 로더는 이 환경변수를 호출한다. 그래서 환경변수에 설정되어 있는 디렉토리가 호출되면 그 디렉토리에 있는 클래스들을 먼저 JVM에 로드한다.
- CLASSPATH 환경변수에는 필수 클래스들이 위치한 디렉토리를 등록하도록 한다.
<img src="https://cys779988.github.io/assets/img/java(5).PNG">

#### java runtime에 -classpath 옵션 사용
-   ```javac <options> <source files>```  
- 컴파일러가 컴파일 하기 위해서 필요로 하는 참조할 클래스 파일들을 찾기 위해서 컴파일시 파일경로를 지정해주는 옵션
- 필요한 클래스 파일들이   ```C:\Java\Engclasses```   에 위치한다면   ```javac -classpath C:\Java\Engclasses C:\Java\Hello.java```  
