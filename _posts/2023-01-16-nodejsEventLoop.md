---
title:  "Node.js Event Loop"
excerpt: Node.js Event Loop
categories:
  - nodejs
---

## libuv
- libuv는 운영체제의 커널을 추상화한 비동기 I/O 라이브러리다.
- 커널이 지원하는 비동기 작업을 libuv에 요청하면 libuv는 커널에게 이 작업을 요청하여 비동기적으로 실행한다.
- 커널이 지원하지 않는 비동기 작업을 libuv에 요청하면 내부에 가지고 있는 스레드 풀에게 이 작업을 요청한다.
- 노드는 I/O 작업을 libuv에게 위임함으로써 논블로킹 I/O를 지원하고 그 기반에는 **이벤트 루프**가 있다.

## 이벤트 루프
- 이벤트 루프는 가능하다면 언제나 시스템 커널에 작업을 떠넘겨서 노드가 논블로킹 I/O 작업을 수행하도록 해준다.
- 현재 대부분의 커널은 멀티 스레드이므로 백그라운드에서 다수의 작업을 실행할 수 있다. 이러한 작업 중 하나가 완료되면 커널이 노드에게 알려주어 적절한 콜백을 poll 큐에 추가하여 실행된다.
- 노드를 시작할 때 이벤트 루프를 초기화하고 제공된 스크립트를 처리한다. 이 때 스크립트는 비동기 API를 호출하거나 스케줄링된 타이머를 사용하거나 `process.nextTick()`을 호출할 수 있다. 그 다음 이벤트 루프 처리를 시작한다.

## 이벤트 루프의 작업 순서

  
```

   ┌───────────────────────────┐
┌─>│           timers          │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │
│  └─────────────┬─────────────┘      ┌───────────────┐
│  ┌─────────────┴─────────────┐      │   incoming:   │
│  │           poll            │<─────┤  connections, │
│  └─────────────┬─────────────┘      │   data, etc.  │
│  ┌─────────────┴─────────────┐      └───────────────┘
│  │           check           │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │
   └───────────────────────────┘


   ┌───────────────────────────┐
   │       nextTickQueue       │
   └───────────────────────────┘
   ┌───────────────────────────┐
   │       microTaskQueue      │
   └───────────────────────────┘

```  

- 자바스크립트를 실행하면 노드는 이벤트 루프를 생성한다. 생성한 이벤트 루프에 진입하지 않고 이벤트 루프 바깥에서 자바스크립트를 끝까지 차례대로 실행한다. 그 후 이벤트 루프에 남아있는 작업이 있는지 확인한다. 남아있는 작업이 있을 때 위의 단계를 차례대로 돌면서 실행할 수 있는 작업을 실행한다. 매 반복마다 이벤트 루프가 살아있는지 확인하고 죽었다면 `exit callbacks`을 실행하고 프로그램을 종료한다.

- 각 단계별로 실행할 콜백의 큐를 가진다. 각 단계는 자신만의 방법에 제한적이므로 보통 이벤트 루프가 해당 단계에 진입하면 해당 단계에 한정된 작업을 수행하고, 큐를 모두 소진하거나 콜백의 최대 개수를 실행할 때까지 해당 단계의 큐에서 콜백을 실행한다. 큐를 모두 소진하거나 콜백 제한에 이르면 이벤트 루프는 다음 단계로 이동한다. 즉, 각 단계는 시스템의 실행 한도의 영향을 받기 때문에 쌓인 작업을 처리하다가 포기하고 다음 단계로 이동한다는 말이다.

- 이벤트 루프가 노드의 비동기 실행을 도와주는 것과 별개로 싱글 스레드이므로 한번에 한 단계에만 진입해 하나의 작업만 수행할 수 있다. poll 단계 작업을 처리하면서 check 단계의 작업을 동시에 처리하거나 poll 단계의 작업을 한번에 여러 개씩 처리하는 것은 불가능하다.

### Timers Phase
- 이벤트 루프의 시작단계로 `setTimeout()`과 `setInterval()`과 같은 타이머 관련 콜백을 실행한다.
- 타이머들이 호출되자마자 이벤트 큐에 들어가는 것이 아니고, 내부적으로 타이머들을 `min-heap` 형태로 유지하고 있다가 실행할 때가 된 타이머들의 콜백을 큐에 넣고 실행한다.
- 노드가 이 단계에 진입해야만 타이머들이 실행될 기회를 얻는다. 따라서 우리가 poll 단계에서 `setTimeout(fn, 1)`을 호출한다 해도 정확히 1ms 뒤에 콜백이 실행됨을 보장하지 않는다.
- 큐에 있는 모든 작업을 실행하거나 시스템의 실행 한도에 다다르면 다음 단계로 넘어간다.
- timers 단계에서 기술적으로는 poll 단계에서 타이머를 언제 실행할지 제어한다.

  
```js
const fs = require('fs');

function someAsyncOperation(callback) {
  // 이 작업이 완료되는데 95ms가 걸린다고 가정합니다.
  fs.readFile('/path/to/file', callback);
}

const timeoutScheduled = Date.now();

setTimeout(() => {
  const delay = Date.now() - timeoutScheduled;

  console.log(`${delay}ms have passed since I was scheduled`);
}, 100);

// 완료하는데 95ms가 걸리는 someAsyncOperation를 실행합니다.
someAsyncOperation(() => {
  const startCallback = Date.now();

  // 10ms가 걸릴 어떤 작업을 합니다.
  while (Date.now() - startCallback < 10) {
    // 아무것도 하지 않습니다.
  }
});
```  

위 예제는 100ms 임계 값 이후에 실행되도록 만료시간을 지정하고, 스크립트는 95ms가 걸리는 파일 읽기를 비동기로 시작한다.

- 이벤트 루프가 poll 단계에 진입했을 때 빈 큐를 가지고 있으므로(`fs.readFile()`이 아직 완료되지 않음.) 가장 빠른 타이머의 임계 값에 도달할 때까지 대기한다.
- `fs.readFile()`이 파일 읽기를 완료하면 10ms가 걸리는 콜백이 poll 큐에 추가되어 실행된다.
- 콜백이 완료되었을 때 큐에 있는 콜백이 없으므로 이벤트 루프는 가장 빠른 타이머가 임계 값에 도달했는지를 확인하고, 타이머의 콜백을 실행하기 위해 timers단계로 돌아간다.
- 타이머가 스케줄링되고 콜백이 실행되기까지 105ms가 걸리는 것을 볼 수 있다.

> libuv는 poll 단계가 이벤트 루프를 모두 차지하는 것을 막기 위해 이벤트 폴링을 멈추게 하는 최댓값(시스템마다 다름)을 가진다.

### Pending Callbacks Phase
- `pending_queue`에 들어있는 콜백들을 실행하는 단계.
- `pending_queue`에는 이전 루프에서 수행되지 못했던 I/O 콜백들이 담겨있다.
- 시스템 실행 한도 제한에 의해 큐에 쌓인 모든 작업을 실행하지 못하고 다음 페이즈로 넘어갈 수 있는데 이 때 처리하지 못한 작업들을 쌓아놓고 실행하는 단계다. 작업들의 예로 네트워크 I/O가 끝나고 응답받은 콜백 또는 에러 핸들러 콜백 등이 있을 수 있다.

### Idle, Prepare Phase
- 이벤트 루프가 매번 순회할 때마다 실행되며 poll 단계를 위한 준비작업 단계.
- 코드의 실행에 직접적인 영향을 미치지 않고 내부적으로만 사용한다.

### Poll Phase
- 대기중인 콜백을 콜 스택으로 가장 많이 올려보내는 단계로 이 단계에서는 새로운 수신 커넥션을 위한 소켓과 데이터를 설정한다.
- poll 단계에서는 `watcher_queue`를 바라보며 작업을 수행하는데 만약 큐가 비어있지 않다면 배정받은 시간동안 큐가 모두 소진될 때까지 모든 콜백을 콜 스택으로 올려 실행시킨다. 단, poll 단계도 시스템 실행 한도의 영향을 받는다.

poll 단계는 두 가지 주요 기능을 가진다.
1. `watcher_queue`에 있는 이벤트를 처리한다.
2. I/O를 얼마나 오래 블록하고 폴링해야 하는지 계산한다.

#### watcher_queue
이벤트 루프에 n개의 열린 소켓을 가지고 있고, n개의 완료되지 않은 요청이 있다. 이 n개의 소켓에 대해 소켓과 메타 데이터를 가진 `watcher`를 관리하는 큐가 `watcher_queue`다. 그리고 각 `watcher`는 `File Descriptor`(네트워크 소켓, 파일 등)를 가지고 있다.  

운영체제가 `File Descriptor`가 준비되었다고 알리면 이벤트 루프는 이에 해당하는 `watcher`를 찾을 수 있고 `watcher`가 맡고 있던 콜백을 실행할 수 있다.

#### Poll Phase Blocking
이전 단계들이 자신이 관리하는 큐만 확인하고 다음 단계로 넘기는 것과 다르게 poll 단계에서는 노드가 다음 단계로 이동해 다시 poll 단계로 올 때까지 실행할 수 있는 작업이 있는지를 고려한다.  

poll 단계에 진입해 콜백들을 실행해 `watcher_queue`가 비게 된다면, 또는 처음부터 `watcher_queue`가 비어있다면 이벤트 루프는 poll 단계에서 잠시 대기할 수 있다.

- 이벤트 루프가 종료되었다면 바로 다음 단계로 넘어간다.
- close callbacks, pending callbacks 단계에서 실행할 작업이 있다면 바로 다음 단계로 넘어간다.
- timers 단계에서 즉시 실행할 수 있는 타이머가 있다면 바로 다음 단계로 넘어간다.
- timers 단계에서 즉시 실행할 수 있는 타이머는 없지만 n초 후에 실행할 수 있는 타이머가 있다면 n초를 기다린 후 다음 단계로 넘어간다.
- 남아있는 타이머가 없다면 poll 단계에서 대기하다가 콜백이 큐에 추가됐을 때 즉시 실행한다.

### Check Phase
- `setImmediate()`의 콜백만을 위한 단계로 `setImmediate()`를 사용하여 수행한 콜백만 이벤트 큐에 쌓이고 콜 스택으로 올라간다.
- `setImmediate()`는 이벤트 루프의 별도 단계에서 실행되는 특수한 타이머이다. poll 단계가 완료된 후 콜백 실행을 스케줄링하는데 libuv API를 사용한다.
- 보통 코드가 실행되면 이벤트 루프는 들어오는 연결, 요청 등을 기다리는 poll 단계에 다다른다. 하지만 콜백이 `setImmediate()`로 스케줄링되었고 poll 단계가 유휴상태가 되었다면 poll 이벤트를 기다리지 않고 check 단계로 넘어가게 된다.

### Close Callbacks Phase
- `socket.on('close', () => {})` 와 같은 close 타입의 콜백을 관리하는 단계.
- 이벤트 루프가 실행되는 사이 노드는 다른 비동기 I/O나 타이머를 기다리고 있는지 확인하고 기다리고 있는 것이 없다면 종료한다.
- 소켓이나 핸들이 갑자기 닫힌 경우( `socket.destroy()` ) 이 단계에서 close 이벤트가 발생한다. 그렇지 않으면 `process.nextTick()`으로 실행될 것이다.

> 윈도우와 Unix/Linux 구현체간에 약간의 차이가 있다. 실제 7~8단계가 있지만 노드가 실제로 사용해서 신경 써야 하는 가장 중요한 단계는 위의 단계다.

### nextTickQueue, microTaskQueue
`nextTickQueue`, `microTaskQueue`는 이벤트 루프의 일부는 아니다. libuv에 포함되어 있지않고 노드에 구현되어 있다. 따라서 이벤트 루프의 단계와 상관없이 동작한다.  

- `nextTickQueue` : `process.nextTick()`의 콜백을 관리.
- `microTaskQueue` : Resolve된 Promise 콜백을 관리.

`nextTickQueue`, `microTaskQueue`는 현재 단계와 상관없이 지금 수행하고 있는 작업이 끝나면 즉시 실행한다. 그 중 `nextTickQueue`는 `microTaskQueue`보다 높은 우선순위를 가지므로 먼저 실행된다.

  
```js
Promise.resolve().then(() => console.log('resolve'))
process.nextTick(() => console.log('nexTick'))
/*
nexTick
resolve
*/
```  

이벤트 루프의 각 단계와 다르게 시스템 실행 한도의 영향을 받지 않는다. 따라서 노드는 큐가 비워질 때까지 콜백들을 실행한다.

  
```js
const fn = () => {
    process.nextTick(fn)
}
​
setTimeout(() => {
    console.log("Timer")
},0)
​
fn()

// Timer는 출력되지 않는다.
```  

#### nextTickQueue, microTaskQueue의 동작 변화
`nextTickQueue`와 `microTaskQueue`, 이벤트 루프 단계간의 동작 순서는 노드의 버전에 따라 다르다. 정확히는 Node v11.0.0을 기점으로 달라졌다. 지금까지의 동작 방식은 Node v11.0.0 이후의 방식이다.  

  
```js
setTimeout(() => {
    console.log(1)
    process.nextTick(() => {
        console.log(3)
    })
    Promise.resolve().then(() => console.log(4))
}, 0)
setTimeout(() => {
    console.log(2)
}, 0)
```  

Node v10.0.0에서는 한 단계에서 다음 단계로 넘어갈 때 `nextTickQueue`와 `microTaskQueue`를 검사한다. 따라서 위 코드의 결과는 `1 2 3 4` 가 된다.  
Node v11.0.0에서는 현재 실행하고 있는 작업이 끝나면 즉시 큐를 검사하고 실행한다. 따라서 위 코드의 결과는 `1 3 4 2`가 된다.



###### Reference

- https://nodejs.org/ko/docs/guides/event-loop-timers-and-nexttick/

- https://www.korecmblog.com/node-js-event-loop/