---
title:  "Java Generic"
excerpt: Java Generic
categories:
  - Java
---

## 제네릭
- 데이터 타입을 일반화 하는 것
- 클래스나 메서드에서 사용할 내부 데이터 타입을 컴파일 시에 미리 지정하는 방법
- 컴파일 시 type check를 함으로써 클래스나 메서드 내부에서 사용되는 객체의 타입의 안정성을 높이고, 반환값에 대한 타입 변환 및 타입 검사에 들어가는 노력을 줄일 수 있다.
- Java 5 이전에는 여러 타입을 사용하는 대부분의 클래스나 메서드에서 인수나 반환값으로 Object 타입을 사용했었다. 이 경우 반환된 Object 객체를 다시 원하는 타입으로 cast 해야하고 이때 오류가 발생할 가능성도 생긴다.
- Java5 부터 도입된 제네릭을 사용하면 컴파일 시에 미리 타입이 정해지므로, 타입검사나 타입변환과 같은 번거로운 작업을 생략할 수 있게 된다.

## 제네릭을 사용하는 이유
- 잘못된 타입이 사용될 수 있는 문제를 컴파일 과정에서 제거
- 실행 시 발생할 수 있는 타입 에러를 컴파일 시 미리 강하게 체크하여 사전에 에러를 방지
- 타입 변환을 할 필요가 없어 프로그램 성능이 향상

## 제네릭 사용법

#### 제네릭 선언 및 생성

  
```java
class DemoGeneric<T> {
    T element;
    void setElement(T element) {
        this.element = element;
    }
    T getElement() {
        return element;
    }
}
```  


#### 타입 변수
- 클래스 명 옆에 <T> 로 DemoGeneric이라는 클래스 내부에서 T를 사용할 수 있게 된다.
- T는 클래스 내부에서 타입 매개변수를 대표하는 값으로 사용된다.
- 타입변수는 어떠한 문자를 사용해도 되지만 네이밍 규칙을 지켜주는 것이 좋다.
- E(element), K(key), N(number), T(type), V(value), R(return type)

  
```java

class GenericTest {

    static class DemoGeneric<E, V> {
        E element;
        V value;
        void setElement(E element) {
            this.element = element;
        }
        E getElement() {
            return element;
        }
        void setValue(V value) {
            this.value = value;
        }
        V getValue() {
            return value;
        }
    }

    public static void main(String[] args) {
        DemoGeneric<String, Integer> gr = new DemoGeneric<>();
        gr.setElement("Test");
        gr.setValue(27);
        System.out.println(gr.getElement());
        System.out.print(gr.getValue());
    }
}

```  


#### 제네릭 메서드 생성

  
```java
public class GenericTest {

    public static <T> T genericMethod(T t) {
        return t;
    }

    public static <T> void printAll(List<T> list) {
        for(T t : list) {
            System.out.print(t);
        }
    }

    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>();
        list.add(1);
        list.add(2);
        list.add(3);

        System.out.println(genericMethod("Test"));
        printAll(list);

    }
}
```  


- 제네릭 클래스에서는 해당 클래스 내부에서 사용할 타입 파라미터가 무엇인지 알려주기 위해 class를 선언할 때 알려주었다면 제네릭 메서드에서는 메서드를 정의할 때, 해당 메서드 내부에서 사용할 타입 파라미터가 무엇인지 알려주기 위해 메서드를 정의할 때 먼저 나열해주고 사용해야 한다. 즉, 리턴타입을 명시하기 전에 작성되어야 한다.


## 바운드 타입 매개변수
- 바운드타입은 특정 타입의 서브타입으로 제한한다. 클래스나 인터페이스를 설계할 때 가장 흔하게 사용된다.

  
```java
public class BoundTypeTest <T extends Number>{
    public void set(T value) {}

    public static void main(String[] args) {
        BoundTypeTest<Integer> boundTypeTest = new BoundTypeTest<>();
        BoundTypeTest.set("Hi");
    }
}
```  

- BoundTypeTest 클래스의 Type 파라미터를 T로 선언하고 <T extends Number> 로 선언하여 BoundTypeTest의 타입으로 Number의 서브타입만 허용한다.
- BoundTypeTest 클래스 내부 메서드 set의 인자에 T 타입을 사용하면 Number의 서브타입만 허용하기 때문에 문자열을 전달하려고 하면 컴파일 에러가 발생한다.


## WildCard
- 제네릭으로 구현된 메서드의 경우 선언된 타입으로만 매개변수를 입력해야한다. 이를 상속받은 클래스 혹은 부모클래스를 사용하고 싶어도 불가능하고 어떤 타입이 와도 상관없는 경우 대응하기 좋지않다. 이를 위해 WildCard를 사용한다.

#### Unbounded WildCard
-   ```List<?>```  와 같은 형태로 물음표만 가지고 정의된다. 내부적으로 Object로 정의되어서 사용되고 모든 타입의 인자를 받을 수 있다.
- 내부적으로 Object로 정의되어서 사용되고 모든 타입의 인자를 받을 수 있다. 타입 파라미터에 의존하지 않는 메서드만을 사용하거나 Object 메서드에서 제공하는 기능으로 충분한 경우에 사용한다.
- Object 클래스에서 제공되는 기능을 사용하여 구현할 수 있는 메서드를 작성하는 경우나 타입 파라미터에 의존하지 않는 일반 클래스의 메서드를 사용하는 경우에 사용됨

#### Upper Bounded WildCard
-   ```<? extends Foo>```  와 같은 형태로 사용하고, 특정 클래스의 자식 클래스만을 인자로 받는다. 임의의 Foo 클래스를 상속받는 어떤 클래스가 와도 되지만 Foo 클래스에 정의된 기능만 사용가능하다.

#### Lower Bounded WildCard
-   ```<? super 하위타입>```  와 같은 형태로 사용하고, Upper Bounded WildCard와 다르게 특정 클래스의 부모 클래스만을 인자로 받는다.
