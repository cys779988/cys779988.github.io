---
title:  "Java Thread"
excerpt: Java Thread
categories:
  - Java
---

## Process
- 단순히 실행중인 프로그램
- 사용자가 작성한 프로그램이 운영체제 의해 메모리 공간을 할당 받아 실행중인 것
- 프로그램에 사용되는 데이터, 메모리 등의 자원과 쓰레드로 구성된다.

## Thread
- 프로세스 내에서 실제로 작업을 수행하는 주체
- 모든 프로세스에는 1개 이상의 쓰레드가 존재하여 작업을 수행
- 경량 프로세스라고 불리며 가장 작은 실행 단위

## Thread 클래스, Runnable 인터페이스
- 자바에서 쓰레드를 생성하는 2가지 방법
- Thread 클래스는 Runable 인터페이스를 구현한 클래스이므로 어떤 것을 적용하느냐의 차이
- 다른 클래스를 확장할 필요가 있을 경우 Runable 인터페이스를 구현하면 되며, 그렇지 않을 경우는 Thread 클래스를 사용하는 것이 편리하다.

#### Thread 클래스
  
```java
  public class Thread implements Runnable {
      private static native void registerNatives();
      static {
          registerNatives();
      }

      ...
  }
```  

#### Runnable 인터페이스
  
```java
  @FunctionalInterface
  public interface Runnable {

      public abstract void run();
  }
```  

## Thread 실행

  
```java
  public class DemoThread extends Thread {
    @Override
    public void run() {
      System.out.println("DemoThread run");
    }
  }
```  

  
```java
  public class DemoRunnable implements Runnable {
    @Override
    public void run() {
      System.out.println("DemoRunnable run");
    }
  }
```  

  
```java
  public class Test {
    public static void main(String[] args) {
      DemoRunnable runnable = new DemoRunnable();
      new Thread(runnable).start();

      DemoThread thread = new DemoThread();
      thread.start();
      System.out.println("end");
    }
  }
```  

  
```
    // 실행결과
    DemoRunnable run
    end
    DemoThread run
```  

- start() 메서드가 끝날 때까지 기다리지 않고, 다음 작업을 실행한다. 새로운 쓰레드를 시작하므로 run() 메서드가 종료될 때까지 기다리지않는다.

  
```java
  // 동시에 여러 쓰레드 실행
  public class Test {
    public static void main(String[] args) {
      DemoRunnable[] runnable = new DemoRunnable[3];
      DemoThread[] thread = new DemoThread[3];

      for (int i = 0; i < 3; i++) {
        runnable[i] = new DemoRunnable();
        thread[i] = new DemoThread();

        new Thread(runnable[i]).start();
        thread[i].start();
      }
      System.out.println("end");
    }
  }
```  
- 쓰레드는 순서대로 실행되지 않는다.
- 컴퓨터의 성능에 따라 달라질 수도 있으며 매번 다른 결과가 나타난다.
- run() 메서드가 끝나지 않으면 애플리케이션은 종료되지 않는다.

## Main 쓰레드
- main 메서드가 실행되는 쓰레드
- 메인 쓰레드는 프로그램이 시작하면 가장 먼저 실행되는 쓰레드이며, 모든 쓰레드는 메인 쓰레드로부터 생성된다.
- 다른 쓰레드를 생성해서 실행하지 않으면, 메인 쓰레드가 종료되는 순간 프로그램도 종료된다. 하지만 여러 쓰레드를 실행하면 메인 쓰레드가 종료되어도 다른 쓰레드가 작업을 마칠 때까지 프로그램이 종료되지 않는다.
- 쓰레드는 사용자 쓰레드와 데몬 쓰레드로 구분되는데 실행중인 사용자 쓰레드가 하나도 없을 때 프로그램이 종료된다.

  
```java
  public class ThreadDemo {

      public static void main(String[] args) {
          Thread t1 = Thread.currentThread();
          System.out.println("currentThread = " + t1);

          Thread t2 = new Thread(new Thread1());
          System.out.println("newThread = " + t2);
      }
  }
  
  class Thread1 implements Runnable {

      @Override
      public void run() {}
  }
```  

  
```
    // 실행결과
    currentThread = Thread[main,5,main]
    newThread = Thread[Thread-0,5,main]
```  

## 쓰레드 그룹
- 서로 관련된 쓰레드는 쓰레드 그룹으로 묶어서 일괄적인 작업 처리를 할 수 있다. 쓰레드 그룹은 다른 쓰레드 그룹을 포함시킬 수 있다.
- 쓰레드 그룹은 보안상의 이유로 도입된 개념으로 자신이 속한 쓰레드 그룹이나 하위 쓰레드 그룹은 변경할 수 있지만 다른 쓰레드 그룹의 쓰레드는 변경할 수 없다.
- 모든 쓰레드는 반드시 하나의 쓰레드 그룹에 속하며, 쓰레드 생성 시 쓰레드 그룹을 지정해주지 않으면 자동으로 main 쓰레드 그룹에 속하게 된다. 쓰레드는 자신을 생성한 쓰레드의 그룹과 우선순위를 상속받기 때문이다.

## 데몬 쓰레드
- 쓰레드의 종류는 일반 쓰레드와 데몬 쓰레드로 나뉜다.
- 데몬 쓰레드는 일반 쓰레드의 보조역할을 수행하는 쓰레드, 일반 쓰레드가 종료되면 데몬 쓰레드는 강제적으로 종료된다. 주로 가비지컬렉터, 자동저장, 화면 자동갱신에 사용된다.
- 일반 쓰레드가 종료되면 같이 종료되기 때문에 일반적으로 무한루프로 구현한다.
- 쓰레드를 생성한 다음 setDaemon(true) 메서드를 호출하면 데몬 쓰레드가 생성된다.
- 데몬 쓰레드가 생성한 쓰레드는 자동으로 데몬 쓰레드가 된다.

## 쓰레드의 상태

상태 | 설명
---- | ----
NEW | 쓰레드가 생성되고 아직 start()가 호출되지 않은 상태
RUNNABLE | 실행 중 또는 실행 가능한 상태
BLOCKED | 동기화블럭에 의해서 일시정지된 상태(lock이 풀릴 때까지 기다리는 상태)
WAITING | 쓰레드가 대기중인 상태
TIMED_WAITING | 특정 시간만큼 쓰레드가 대기중인 상태
TERMINATED | 쓰레드가 종료된 상태

- 쓰레드의 상태는 메서드를 통해 제어할 수 있다.
  
메서드 | 설명
---- | ----
static void sleep(long millis) <br/> static void sleep(long millis, int nanos) | 지정된 시간동안 쓰레드를 일시정지. 지정한 시간이 지나고 나면, 자동적으로 다시 실행대기 상태가 된다.
void join()  <br/> void join(long millis)  <br/> void join(long millis, int nanos) | 지정된 시간동안 쓰레드가 실행. join()을 호출한 쓰레드는 그동안 일시정지 상태가 된다. 지정된 시간이 지나거나 작업이 종료되면 join()을 호출한 쓰레드로 다시 돌아와 실행을 계속한다.
void interrupt() | sleep()이나 join()에 의해 일시정지 상태인 쓰레드를 깨워서 실행대기 상태로 만든다. 해당 쓰레드에서는 InterruptedException이 발생함으로써 일시정지 상태를 벗어나게 된다.
static void yield() | 실행 중에 자신에게 주어진 실행시간을 다른 쓰레드에게 양보하고 자신은 실행대기 상태가 된다.
void stop() | 쓰레드를 즉시 종료.
void suspend() | 쓰레드를 일시정지. resume()을 호출하면 다시 실행대기 상태가 된다.
void resume() | suspend()에 의해 일시정지 상태에 있는 쓰레드를 실행대기 상태로 만든다.

- stop(), suspend(), resume() 은 쓰레드를 교착상태로 만들기 쉽기 때문에 deprecated 되었다.


<img src="https://cys779988.github.io/assets/img/java(7).png">

#### I/O Blocking
- 사용자 입력을 받을 때는 사용자 입력이 들어오기 전까지 해당 쓰레드가 일시정지 상태가 된다. 이를 I/O 블로킹이라고 한다.
- 한 쓰레드 내에서 사용자 입력을 받는 작업과 이와 관련 없는 작업 두 가지 코드를 작성하면, 사용자 입력을 기다리는 동안 다른 작업 또한 중지되기 때문에 CPU의 사용 효율이 떨어진다.
- 사용자 입력 받는 쓰레드와, 이와 관련 없는 쓰레드를 분리하여 더욱 효율적으로 CPU를 사용할 수 있다.

#### 싱글 쓰레드
  
```java
  public class ThreadDemo {
    public static void main(String[] args) {
      String input = JOptionPane.showInputDialog("아무값이나 입력하세요");
      System.out.println("입력 값은 " + input + " 입니다.");

      for (int i = 10; i > 0; i--) {
        System.out.println(i);

        try {
          Thread.sleep(1000);
        } catch (Exception e) {
        }
      }
    }
  }
```  

#### 멀티 쓰레드
  
```java
  public class ThreadDemo {
    public static void main(String[] args) {
      Thread t = new Thread(new MyThread());
      t.start();

      String input = JOptionPane.showInputDialog("아무값이나 입력하세요");
      System.out.println("입력 값은 " + input + " 입니다.");
    }
  }

  class MyThread implements Runnable {
    @Override
    public void run() {
      for (int i = 10; i > 0; i--) {
        System.out.println(i);

        try {
          Thread.sleep(1000);
        } catch (Exception e) {
        }
      }
    }
  }
```  

## 쓰레드의 우선순위
- 쓰레드는 우선순위(priority)라는 멤버변수를 갖고 있다.
- 각 쓰레드 별로 우선순위를 다르게 설정해줌으로써 어떤 쓰레드에 더 많은 작업 시간을 부여할 것인가를 설정해줄 수 있다.
- 1~10 사이의 값을 지정해줄 수 있으며 기본값은 5

  
```java
public class Thread implements Runnable {
    
    public final static int MIN_PRIORITY = 1;

    public final static int NORM_PRIORITY = 5;

    public final static int MAX_PRIORITY = 10;
    
    public final void setPriority(int newPriority) {
        ThreadGroup g;
        checkAccess();
        if (newPriority > MAX_PRIORITY || newPriority < MIN_PRIORITY) {
            throw new IllegalArgumentException();
        }
        if((g = getThreadGroup()) != null) {
            if (newPriority > g.getMaxPriority()) {
                newPriority = g.getMaxPriority();
            }
            setPriority0(priority = newPriority);
        }
    }
    
    public final int getPriority() {
        return priority;
    }
}

```  

- set Priority 메서드는 쓰레드를 실행하기 전에만 호출할 수 있다.
- 쓰레드의 우선 순위를 높이면 더 많은 실행 시간과 실행 기회를 부여받을 수 있다. 주의할 점은 이것이 반드시 보장되지 않는다.
- 쓰레드의 작업할당은 OS의 스케쥴링 정책과 JVM의 구현에 따라 다르기 때문에 코드에서 우선순위를 지정하는 것은 단지 희망사항일 뿐, 실제 작업은 설정한 우선 순위와 다르게 진행할 수 있다.

## 동기화(Synchronization)

#### synchronized
- 멀티 쓰레드 프로세스에서는 여러 프로세스가 메모리를 공유하기 때문에 한 쓰레드가 작업하던 부분을 다른 쓰레드가 간섭하는 문제가 생길 수 있다.
- 어떤 쓰레드가 진행 중인 작업을 다른 쓰레드가 간섭하지 못하도록 하는 작업을 동기화라고 한다.
- 동기화를 하려면 다른 쓰레드가 간섭해서는 안 되는 부분을 synchronized 키워드를 사용하여 임계영역(critical section)으로 설정해 주어야 한다.

  
```java
  // 메서드 전체를 임계영역으로 설정
  public synchronized void method1 () {
      ......
  }
```  

- 쓰레드는 synchronized 키워드가 붙은 메서드가 호출된 시점부터 해당 메서드가 포함된 객체의 lock을 얻어 작업을 수행하다가 메서드가 종료되면 lock을 반환한다.

  
```java
  // 특정한 영역을 임계영역으로 설정
  synchronized(객체의 참조변수) {
      ......
  }
```  

- 참조변수는 락을 걸고자 하는 객체를 참조하는 것이어야 한다.
- 이 영역으로 들어가면서부터 쓰레드는 지정된 객체의 lock을 얻게되고 블록을 벗어나면 lock을 반납한다.

#### lock
- lock은 일종의 자물쇠 개념으로 모든 객체는 lock을 하나식 가지고 있다.
- 해당 객체의 lock을 가지고 있는 쓰레드만 임계영역의 코드를 수행할 수 있다.
- 한 객체의 lock은 하나밖에 없기 때문에 다른 쓰레드들은 lock을 얻을 때까지 기다리게 된다.
- 임계영역은 멀티쓰레드 프로그램의 성능을 좌우하기 때문에 가능하면 메서드 전체에 lock을 거는 것보다 synchronized 블록으로 임계영역을 최소화하는 것이 좋다.

  
```java
  public class Account {
    private int balance = 1000;

    public int getBalance() {
      return balance;
    }

    public void withdraw(int money) {
      // 동기화블록
      synchronized (this) {
        if(balance >= money) {
          try {
            Thread.sleep(1000);
          } catch (InterruptedException e) {
          }
          balance -= money;
        }
      }
    }
  }
```  

- 출금하는 로직에 동기화를 해서 한 쓰레드가 출금로직을 실행하고 있으면 다른 쓰레드가 출금블록에 들어오지 못하도록 막아줌

  
```java
  public class ThreadDemo implements Runnable	{
    Account account = new Account();

    @Override
    public void run() {
      // TODO Auto-generated method stub
      while (account.getBalance() > 0) {
        int money = (int) (Math.random() * 3 + 1) * 100;
        account.withdraw(money);
        System.out.println("balance : " + account.getBalance());
      }
    }
  }
```  

  
```java
  public class Main {
    public static void main(String[] args) {
      Runnable r = new ThreadDemo();
      new Thread(r).start();
      new Thread(r).start();
    }
  }
```  

#### 교착상태(DeadLock)
- 교착상태는 한 자원을 여러 시스템이 사용하려고 할 때 발생

<img src="https://cys779988.github.io/assets/img/java(8).png">

- Process1 과 Process2가 모두 자원 A, B가 필요한 상황에서 Process1은 A에 먼저 접근하고 Process2는 B에 먼저 접근했다.
- Process1과 Process2는 각각 A와 B의 lock을 가지고 있는 상태
- Process1은 B에 접근하기 위해 B의 lock이 풀리기를 대기하고 Process2는 A에 접근하기 위해 A의 lock이 풀리기를 대기한다.
- 서로 원하는 리소스가 상대방에게 할당되어 있기 때문에 두 프로세스는 무한히 대기상태에 있게 되는데, 이를 교착상태라 한다.

교착상태는 한 시스템 내에서 다음의 네 가지 조건이 동시에 성립될 때 발생한다. 아래 네 가지 조건 중 하나라도 성립하지 않도록 만들면 교착상태를 해결할 수 있다.

1. 상호배제(Mutual exclusion) : 자원은 한 번에 한 프로세스만이 사용할 수 있어야 한다.
2. 점유대기(Hold and wait) : 최소한 하나의 자원을 점유하고 있으면서 다른 프로세스에 할당되어 사용하고 있는 자원을 추가로 점유하기 위해 대기하는 프로세스가 있어야 한다.
3. 비선점(No preemption) : 다른 프로세스에 할당된 자원은 사용이 끝날 때까지 강제로 빼앗을 수 없어야 한다.
4. 순환대기(Circular wait) : 프로세스의 집합(P0, P1, ..., Pn)에서 P0는 P1이 점유한 자원을 대기하고 P1은 P2가 점유한 자원을 대기하고 ... Pn-1은 Pn이 점유한 자원을 대기하며 Pn은 P0가 점유한 자원을 요구해야 한다.

#### wait() & notify()
- 동기화를 하게 되면 하나의 작업을 하나의 쓰레드로만 처리하기 때문에 작업 효율이 떨어진다. 이때 동기화의 효율을 높이기 위해서 wait(), notify()를 사용한다.


메서드 | 설명
---- | ----
void wait() <br/> void wait(long timeout) <br/> void wait(long timeout, int nanos) | 객체의 lock을 풀고 쓰레드를 해당 객체의 waiting pool에 넣는다.
void notify() | waiting pool에서 대기 중인 쓰레드 하나를 깨운다.
void notifyAll() | waiting pool에서 대기 중인 모든 쓰레드를 깨운다.

- wait(), notify()는 Object 클래스에 정의되어 있으며, 동기화 블록 내에서만 사용할 수 있다.
- 동기화된 임계 코드 영역의 작업을 수행하다가 작업을 더 이상 진행할 상황이 아니면, 일단 wait()을 호출하여 쓰레드가 lock을 반납하고 기다리게 한다.
- 다른 쓰레드가 lock을 얻어서 해당 객체에 대한 작업을 수행한다.
- 나중에 작업을 진행할 수 있는 상황이 되면 notify()를 호출하여 작업을 중단했던 쓰레드가 다시 lock을 얻어 작업을 진행할 수 있게 된다.

  
```java
  public class Account {
    private int balance = 1000;

    public synchronized void withdraw(int money) {
      while (balance < money) {
        try {
          wait();
        } catch (InterruptedException e) {
        }
      }
      balance -= money;
    }

    public synchronized void deposit(int money) {
      balance += money;
      notify();
    }
  }
```  

- 잔고가 모자라서 출금을 할 수 없는 경우, 다른 쓰레드가 입금 할 수 있도록 wait() 메서드를 호출하여 객체에 대한 lock을 풀고 waiting pool 에서 기다린다.
- deposit을 수행하는 쓰레드는 해당 객체의 lock을 얻어 잔고를 채우고 notify() 메서드를 호출하여 waiting pool 에서 대기중인 쓰레드에게 다시 작업을 수행하라고 통보한다.
- 대기중이던 쓰레드는 다시 락을 얻어 인출 로직을 수행한다.

#### java.util.concurrent.locks
- JDK 1.5부터 synchronized 외에 java.util.concurrent.locks 패키지의 Lock 클래스들을 사용하여 동기화를 구현
- synchronized로 동기화를 하면 자동으로 lock이 걸리고 풀리지만 같은 메서드 내에서만 lock을 걸 수 있다는 불편함이 있다. 그럴 때 lock 클래스를 이용한다.

  
```java

public interface Lock {}

// 읽기에는 공유적이고, 쓰기에는 배타적인 lock
public class ReentrantLock implements Lock, java.io.Serializable {}

// 재진입이 가능한 lock, 가장 일반적인 배타 lock
public class ReentrantReadWriteLock implements ReadWriteLock, java.io.Serializable {}

// ReentrantReadWriteLock에 낙관적인 lock의 기능을 추가
public class StampedLock implements java.io.Serializable {}
```  

- ReentrantLock : 가장 일반적인 lock. 특정 조건에서 lock을 풀었다가 다시 lock을 걸 수 있다.
- ReentrantReadWriteLock : 읽기를 위한 lock(ReadLock)과 쓰기를 위한 lock(WriteLock)을 제공. ReentrantLock은 무조건 lock이 있어야만 임계영역의 코드를 수행할 수 있지만, ReentrantReadWriteLock은 읽기 lock이 걸려 있으면, 다른 쓰레드가 읽기 lock을 중복해서 걸고 읽기를 수행할 수 있다. 그러나 읽기 lock이 걸린 상태에서 쓰기 lock은 허용되지않는다. 반대의 경우도 동일하다.
- StampedLock : lock을 걸거나 해지할 때 Stamp(Long 타입 정수)를 사용하며 ReentrantReadWriteLock에 optimistic reading lock이 추가된 형태. 읽기 lock이 걸려 있으면 쓰기 lock을 얻기 위해서는 읽기 lock이 풀릴 때까지 기다려야 하는데 비해 optimistic reading lock은 쓰기 lock에 의해 바로 풀린다.
