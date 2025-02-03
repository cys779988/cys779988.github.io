---
title:  "병렬프로그래밍 Executor Framework"
excerpt: 병렬프로그래밍 Executor Framework
categories:
  - Java
---


작업(Task)은 논리적인 업무의 단위이며, 스레드는 특정 작업을 비동기적으로 동작시킬 수 있는 방법을 제공한다. 순차적인 방법은 응답 속도와 전체적인 성능이 크게 떨어지는 문제점이 있고, 작업별로 스레드를 만들어내는 방법은 자원관리 측면에서 허점이 있다.  

## Thread 예제

#### 작업을 순차적으로 실행
  
```java
class SingleThreadWebServer {
	public static void main(String[] args) {
		ServerSocket socket = new ServerSocket(80);
		while (true) {
			Socket conn = socket.accept();
			handleRequest(conn);
		}
	}
}
```  

위 경우 단일 스레드에서 IO작업을 하는동안 CPU가 대기하고 있어야하는 경우가 있고 하드웨어 자원을 효율적으로 활용하지 못하는 문제가 있다.

#### 작업마다 스레드를 생성

  
```java
class ThreadPerTaskWebServer {
	public static void main(String[] args) {
		ServerSocket socket = new ServerSocket(80);
		while (true) {
			final Socket conn = socket.accept();
			Runnable task = new Runnable() {
				public void run() {
					handleRequest(conn);
				}
			};
			new Thread(task).start();
		}
	}
}
```  

요청이 들어올 때마다 새로운 스레드를 생성한다. 클라이언트의 요청 처리를 메인 스레드가 아닌 새로운 스레드에서 처리한다.


## 스레드를 많이 생성할 때 문제점
#### 스레드 라이프 사이클
생성과 제거 작업에도 자원이 소모된다. 따라서 클라이언트의 요청을 처리할 때 딜레이가 생기고 JVM과 운영체제는 몇 가지 기초적인 작업을 수행한다.

#### 자원낭비
실행중인 Java Thread는 시스템자원, 특히 메모리를 소모한다. 프로세서보다 많은 스레드를 만들어 동작중이라면 대부분의 스레드는 idle 상태에 머무르게 된다. 대기 상태에 머무르는 스레드가 많아질수록 많은 메모리가 필요하며, GC에 가해지는 부하가 늘어날 뿐만 아니라 CPU 사용을 위해 여러 자바 스레드가 경쟁상태에 빠지게 되며 메모리 외에도 많은 자원을 소모한다.

#### 안정성
모든 시스템에는 생성할 수 있는 스레드의 개수가 제한되어 있다. Thread 클래스에 필요한 스택의 크기가 제한된 양을 초과하게 되면 OutOfMemoryError가 발생한다.


## Executor

자바 클래스 라이브러리에서 작업를 실행하고자 할 때 Thread 보다 Executor 가 훨씬 추상화가 잘 되어 있어서 사용하기 좋다.

  
```java
public interface Executor {
    void execute(Runnable command);
}
```  

Executor는 단순한 인터페이스로 보이지만, 다양한 종류의 작업 실행 정책을 지원하는 비동기적 작업 실행 프레임워크의 근간을 이루는 인터페이스다. Executor는 작업 등록과 작업 실행을 분리하는 표준적인 방법이며, 각 작업은 Runnable의 형태로 정의한다. Executor 인터페이스를 구현한 클래스는 작업의 라이프 사이클을 관리하는 기능도 갖고 있고, 통계 값을 뽑아내거나 또는 애플리케이션에서 작업 실행 과정을 관리하고 모니터링하기 위한 기능도 갖고 있다.  

Executor의 구조는 producer-consumer 패턴에 기반하고 있으며, 작업을 생성해 등록하는 클래스가 프로듀서가 되고 작업을 실제로 실행하는 스레드가 컨슈머가 된다.


#### Executor 사용 예
  
```java
class TaskExecutionWebServer {
	private static final int NTHREADS = 100;
	private static final Executor exec = Executors.newFixedThreadPool(NTHREADS);

	public static void main(String[] args) throws IOException {
		ServerSocker socker = new ServerSocker(80);
		while (true) {
			final Socket connection = socket.accept();
			Runnable task = new Runnable() {
				public void run() {
					handleRequest(connection);
				}
			};
			exec.execute(task);
		}
	}
}
```  

요청 처리 작업을 등록하는 부분과 실제로 처리 기능을 실행하는 부분이 Executor를 사이에 두고 분리되어 있고, Executor를 다른 방법으로 구현한 클래스를 사용하면 비슷한 기능에 다른 특성으로 동작하도록 손쉽게 변경할 수 있다. 스레드를 직접 생성하도록 구현되어 있는 상태에서는 서버의 동작 특성을 쉽게 변경할 수 없었지만, Executor를 사용하면 Executor의 설정을 변경하는 것만으로 쉽게 변경된다. Executor에 필요한 설정은 대부분 초기에 한 번 지정하는 것이 보통이며 처음 실행하는 시점에 설정 값을 지정하는 편이 좋다.

#### Executor 상속

  
```java
public class ThreadPerTaskExecutor implements Executor {
//    // 작업을 순차적으로 처리
//    public void execute(Runnable r) {
//        r.run();
//    };

    // 각 작업마다 새로운 스레드 할당
    public void execute(Runnable r) {
        new Thread(r).start();
    };
}
```  


## 실행 정책
작업을 등록하는 부분과 실행하는 부분을 서로 분리시켜두면 특정 작업을 실행하고자 할 때 코드를 많이 변경하거나 기타 여러 가지 어려운 상황에 맞닥뜨리지 않으면서 실행 정책을 쉽게 변경할 수 있는 장점이 있다.

#### 실행 정책 예
- 작업을 어느 스레드에서 실행할 것인가?
- 작업을 어떤 순서로 실행할 것인가?
- 동시에 몇 개의 작업을 병렬로 실행할 것인가?
- 최대 몇 개까지의 작업이 큐에서 실행을 대기할 수 있게 할 것인가?
- 시스템 부하가 많이 걸려서 작업을 거절해야할 경우, 어떤 작업을 희생야으로 삼아야 할 것이며, 작업을 요청한 프로그램에 어떻게 알려야 할 것인가?
- 작업을 실행하기 직전이나 실행한 직후에 어떤 동작이 있어야 하는가?

실행 정책은 일종의 자원 관리 도구라 할 수 있다. 가장 최적화된 실행 정책을 찾으려면 하드웨어나 소프트웨어적인 자원을 얼마나 확보할 수 있는지 확인해야 하고, 더불어 애플리케이션의 성능과 반응 속도가 요구사항에 얼마만큼 명시되어 있는지도 알아야 한다.  

실행 정책과 작업 등록 부분을 명확하게 분리시켜 두면 애플리케이션을 실제 상황에 적용하려 할 때 설치할 하드웨어나 기타 자원의 양에 따라 적절한 실행 정책을 임의로 지정할 수 있다.


## 스레드 풀
작업을 처리할 수 있는 동일한 형태의 스레드를 풀의 형태로 관리한다. 일반적으로 스레드 풀은 풀 내부의 스레드로 처리할 작업을 쌓아둬야 하기 때문에 작업 큐와 밀접한 관련이 있다. 작업 스레드는 아주 간단한 주기로 동작하는데, 먼저 작업 큐에서 실행할 다음 작업을 가져오고, 작업을 실행하고, 실행할 다음 작업이 나타날 때까지 대기하는 일을 반복한다.

### 스레드 풀 장점
매번 스레드를 생성하는 대신 이전에 사용했던 스레드를 재사용하기 때문에 스레드를 계속 생성할 필요가 없다. 따라서 여러 개의 요청을 처리하는데 필요한 시스템 자원이 줄어들게 된다.  
클라이언트가 요청을 보냈을 때 해당 요청을 처리할 스레드가 이미 만들어진 상태로 대기하고 있기 때문에 작업을 실행하는 데 딜레이가 발생하지 않아 전체적인 반응 속도도 향상된다.  

### Executors 클래스의 스레드풀 메서드

#### newFixedThreadPool
  
```java
newFixedThreadPool(int nThreads)
```

처리할 작업이 등록되면 그에 따라 실제 작업할 스레드를 하나씩 생성한다. 생성할 수 있는 스레드의 최대 개수는 제한되어 있으며 제한된 개수까지 스레드를 생성하고 스레드수를 유지한다.(스레드가 작업하는 도중 예상치 못한 예외가 발생하여 스레드가 종료되면 하나씩 더 생성하기도 한다)

#### newCachedThreadPool
  
```java
newCachedThreadPool()
```  

현재 풀에 갖고 있는 스레드의 수가 처리할 작업의 수보다 많아서 쉬는 스레드가 많이 발생할 때 쉬는 스레드를 종료시켜 유연하게 대응할 수 있다. 처리할 작업의 수가 많아진다면 필요한 만큼 스레드를 새로 생성한다. 스레드의 수에 제한을 두지 않는다.

#### newSingleThreadExecutor
  
```java
newSingleThreadExecutor()
```  

단일 스레드로 동작하는 Executor로서 작업을 처리하는 스레드가 단 하나 뿐이다. 만약 작업 중 Exception이 발생하여 비정상적으로 종료되면 새로운 스레드를 생성하여 나머지 작업을 실행한다. 등록된 작업은 설정된 큐에서 지정하는 순서(FIFO, LIFO, 우선순위)에 따라 반드시 순차적으로 처리된다.  

단일 스레드로 실행되는 Executor 역시 내부적으로 충분한 동기화 기법을 적용하고 있다. 따라서 특정 작업이 진행되는 동안 메모리에 남겨진 기록을 다음에 실행되는 작업에서 가져다 사용할 수 있다. 이것은 여러 작업이 모두 하나의 스레드에 제한된 상태로 실행되기 때문에 간혹 작업 실행 스레드가 종료되어 새로운 스레드를 만들어 실행하는 경우에도 똑같이 적용된다.


#### newScheduledThreadPool
  
```java
newScheduledThreadPool(corePoolSize)
```  

일정 시간 이후에 실행하거나 주기적으로 작업을 실행할 수 있다.

newFixedThreadPool과 newCachedThreadPool 팩터리 메서드는 일반화된 형태로 구현되어 있는 ThreadPoolExecutor 클래스의 인스턴스를 생성한다. 생성된 ThreadPoolExecutor 인스턴스에 설정 값을 조절해 필요한 형태를 갖추고 사용할 수도 있다.  

작업별로 스레드를 생성하는 전략(thread-per-task)에서 풀을 기반으로 하는 전략으로 변경하면 안정성 측면에서 엄청난 장점을 얻을 수 있다. 웹서버에 부하가 걸리더라도 메모리가 부족해 스레드를 계속 생성하느라 메모리가 모자란 현상이 발생하지 않기 때문이다. 물론 요청을 처리하는 속도보다 요청이 새로 추가되는 속도가 더 빠르다면 쌓여있는 작업의 수가 계속 늘어나면서 메모리가 모자라 웹서버가 다운될 수 있다. 이런 경우에는 작업 큐의 크기를 제한하는 것으로 해결할 수 있다.  
부하에 따라 수천 개의 스레드를 생성해 제한된 양의 CPU와 메모리 자원을 서로 사용하려고 경쟁시키는 상황에 이르지 않기 때문에 성능이 떨어질 때도 점진적으로 서서히 떨어지는 특징을 갖는다.  

Executor를 사용하면 사용하지 않을 때보다 성능을 튜닝하거나, 실행 과정을 관리하거나, 실행 상태를 모니터링하거나, 실행 기록을 로그로 남기거나, 오류가 발생했을 때 효과적으로 처리할 수 있다.

## Executor 동작 주기
Executor를 구현하는 클래스는 대부분 작업을 처리하기 위한 스레드를 생성하도록 되어있다. 하지만 JVM은 모든 스레드가 종료되기 전에는 종료하지 않고 대기하기 때문에 Executor를 제대로 종료시키지 않으면 JVM 자체가 종료되지 않고 대기할 수도 있다.  

Executor는 작업을 비동기적으로 실행하기 때문에 앞서 실행시켰던 작업의 상태를 특정 시점에 정확하게 파악하기 어렵다. 어떤 작업은 이미 완료됐을 수도 있고, 아직 실행 중일수도 있고,  아직 큐에서 대기 상태에 머물러 있을 수도 있기 때문이다.

#### ExecutorService 인터페이스의 동작 주기 관리 메서드

  
```java
public interface ExecutorService extends Executor {
    void shutdown();

    List<Runnable> shutdownNow();

    boolean isShutdown();

    boolean isTerminated();

    boolean awaitTermination(long timeout, TimeUnit unit)
        throws InterruptedException;

    ...
}
```  

내부적으로 ExecutorService가 가지고 있는 동작 주기는 실행중, 종료중, 종료 세 가지 상태가 있다.

- 실행중(running) : ExecutorService를 처음 생성했을 때
- 종료중(shutting down) : 어느 시점에 shutdown 메서드를 실행하면 안전한 종료 절차를 진행하며 종료중 상태로 전환, shutdownNow 메서드를 실행하면 강제 종료 절차를 진행
- 종료(terminated) : 스레드 종료

일반적으로 shutdown 메서드를 실행한 후 바로 awaitTermination 메서드를 호출하면 마치 ExecutorService를 직접 종료시키는 것과 비슷한 효과를 얻을 수 있다.


## Callable, Future

### Callable
`Runnable.run()` 메서드의 실행이 끝나면 결과 값을 리턴해 줄 수 없고, 예외가 발생해도 throws 구문으로 표현할 수 없다는 단점이 있다. Callable은 단점을 개선하여 call 메서드를 실행 후 결과값을 돌려받을 수 있고 Exception도 발생시킬 수 있다.

### Future
`Callable.call()` 메서드가 `ExecutorService.submit()` 메서드에 전달될 때 곧바로 실행되는 것이 아니기 때문에 리턴값을 바로 구할 수 없다. 이 리턴값은 미래의 어느 시점에 구할 수 있는데, 이렇게 미래에 실행되는 Callable의 수행 결과 값을 구할 때 Future를 사용한다.

  
```java
public class Main {
    public static void main(String[] args) 
    throws InterruptedException, ExecutionException {
        ExecutorService executorService = Executors.newSingleThreadExecutor();

        Callable<String> hello = () -> {
            Thread.sleep(2000L);
            return "Hello";
        };

        Future<String> helloFuture = executorService.submit(hello);
        System.out.println("Started!");

        helloFuture.get(); // blocking call

        System.out.println("End!!");
        executorService.shutdown();
    }
}
```  
