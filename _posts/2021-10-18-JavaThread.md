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
  
```
  public class Thread implements Runnable {
      private static native void registerNatives();
      static {
          registerNatives();
      }

      ...
  }
```  

#### Runnable 인터페이스
  
```
  @FunctionalInterface
  public interface Runnable {

      public abstract void run();
  }
```  

## Thread 실행

  
```
  public class DemoThread extends Thread {
    @Override
    public void run() {
      System.out.println("DemoThread run");
    }
  }
```  

  
```
  public class DemoRunnable implements Runnable {
    @Override
    public void run() {
      System.out.println("DemoRunnable run");
    }
  }
```  

  
```
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

  
```
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

  
```
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
