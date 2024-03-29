---
title:  "Node.js EventLoop와 WorkerPool"
excerpt: Node.js EventLoop와 WorkerPool
categories:
  - nodejs
---

## 노드의 확장성
- 노드는 자바스크립트 코드를 **이벤트 루프(메인 루프, 메인 스레드, 이벤트 스레드 등)** 에서 실행시키고 파일 I/O와 같은 비싼 작업은 **워커 풀(스레드 풀)** 에게 위임하는 **Event-Driven Architecture**로 설계되어 있다.
- 어떤 경우에는 Apache와 같은 서버를 더 많이 두는 것보다 노드가 좋을 정도로 노드의 확장성은 뛰어나다.
- 노드 확장성의 핵심은 많은 수의 클라이언트를 단 몇 개의 스레드만으로 처리한다는 점이다. 노드가 몇 개의 스레드만으로 잘 처리한다는 것은 곧 스레드를 생성하면서 발생하는 메모리 오버헤드와 컨텍스트 스위칭에서 발생하는 오버헤드를 클라이언트의 요청을 처리하는 데에 사용할 수 있다는 의미이기도 하다. 반대로 말하면 우리는 애플리케이션이 몇 개의 스레드만으로 제대로 동작하도록 설계해야한다는 뜻이기도 하다. 즉, 노드가 클라이언트에 관련된 일을 하는 시간이 적어야 한다.


## 이벤트 루프에서 동작하는 코드
- 처음 실행될 때 노드 애플리케이션에서는 초기화 단계를 거친다. 해당 단계에서는 모듈들을 `require`하고 이벤트를 위한 콜백을 등록한다.
- 그 다음 노드 애플리케이션은 이벤트 루프에 진입하고 적절한 콜백을 실행하여 클라이언트의 요청에 응답한다.
- 해당 콜백은 동기적으로 실행되며 해당 콜백이 완료되고 나서 비동기 요청들이 더 등록될 수 있다. 비동기 요청들을 처리하기 위한 콜백 또한 이벤트 루프에서 실행된다.
- 네트워크 I/O처럼 콜백에 의해 계속 요청되는 논블로킹 비동기 요청들도 이벤트 루프가 처리한다.

> 요약하면 이벤트 루프에서는 이벤트를 위해 등록된 자바스크립트 콜백을 실행하고 네트워크 I/O와 같은 논블로킹 비동기 요청들을 처리한다.


## 워커 풀에서 동작하는 코드
- 노드에서 워커 풀은 libuv로 구현되어 있으며 libuv는 일반적인 작업 요청 API를 노출하고 있다.
- 노드는 워커 풀을 값비싼 작업을 처리하는 경우에 사용한다. 해당 작업에는 `CPU-intensive`한 작업들뿐만 아니라 OS에서 논블로킹을 지원하지 않는 I/O도 포함된다.

#### I/O-intensive 모듈 API
- DNS : `dns.lookup()`, `dns.lookupService()`
- File System : `fs.FSWatcher`와 libuv의 스레드 풀을 동기적으로 사용하는 경우를 제외한 모든 파일 시스템 API

#### CPU-intensive 모듈 API
- Crypto : `crypto.pbkdf2()`, `crypto.scrypt()`, `crypto.randomBytes()`, `crypto.randomFill()`, `crypto.generateKeyPair()`
- Zlib : libuv의 스레드 풀을 동기적으로 사용하는 경우를 제외한 모든 zlib API


> 대부분 노드 애플리케이션에서 워커 풀을 사용하는 방법은 위의 API 목록이 유일하다. `C++ add-on`을 사용하는 애플리케이션이나 모듈의 경우 다른 작업을 워커 풀에서 처리할 수 있다.

이벤트 루프의 콜백에 의해 위 목록 중 하나의 API가 호출되었을 때 이벤트 루프에서는 해당 API를 위해 `Node.js C++` 바인딩에 들어가고 워커 풀에 작업을 요청하므로 셋업할 때 약간의 리소스가 사용된다. 하지만 전체 비용에 비하면 무시할 정도이며 이것이 이벤트 루프가 offloading 한 이유다. 노드는 워커 풀에 이러한 작업을 요청할 때 `Node.js C++` 바인딩에서 해당하는 `C++` 함수에 대한 포인터를 함께 제공한다.


## 이벤트 루프와 워커 풀의 코드 실행 흐름
이벤트 루프와 워커 풀에서는 각각 대기 중인 이벤트와 대기 중인 작업을 관리하기 위한 큐를 가지고 있다. 하지만 이벤트 루프는 실제로 큐를 가지고 있지 않다. 그 대신 OS에게 모니터링을 요청하는 FD(File-Descriptor) 들의 콜렉션을 가지고 있으며 이는 epoll(Linux), IOCP(Windows)와 같은 메커니즘으로 동작한다.  
이 FD들은 그것이 모니터링하고 있는 모든 네트워크 소켓, 모니터링 중인 파일 등과 작용한다. OS에서 FD가 준비되었다고 알리면 이벤트 루프에서는 이를 적절한 이벤트로 번역한 후 해당 이벤트에 관련된 콜백을 호출한다.  
이와 반대로 워커 풀에서는 진짜 큐를 사용하여 처리할 작업을 관리한다. 하나의 워커는 하나의 작업을 해당 큐에서 pop해서 처리하며 작업이 완료되면 "최소한 하나의 작업은 끝났음" 이벤트를 이벤트 루프에 보낸다.


## 애플리케이션 디자인
Apache와 같은 하나의 스레드에서 하나의 클라이언트를 처리하는 디자인에서 각각의 대기 중인 클라이언트는 해당 스레드에 할당된다. 만약 하나의 스레드가 하나의 클라이언트에 의해 막히게 되면 OS에서 이를 중단시키고 다른 클라이언트가 사용할 수 있게한다. OS는 적은 양의 작업이 필요한 클라이언트가 더 많은 양의 작업이 필요한 클라이언트에 의해 피해 보지 않도록 조치 한다.  

노드는 많은 클라이언트를 몇 개의 스레드로 처리해야하므로 스레드가 한 클라이언트의 요청에 의해 막힌다면 그 뒤에 대기 중인 클라이언트들의 요청은 스레드가 해당 콜백이나 작업을 끝낼 때까지 대기해야 한다. 그러므로 하나의 콜백이나 작업을 요청하는 클라이언트에 너무 많은 시간을 사용하지 말아야 한다. 이것은 노드가 확장성이 좋은 이유 중 하나지만 한편으로는 우리에게 공정한 스케줄링을 보장해야 한다는 책임이 주어진다는 의미다.


## 이벤트 루프를 막지말자
이벤트 루프에서 콜백을 실행하거나 워커에서 태스크를 실행할 때 오랜 시간이 걸린다면 우리는 그 스레드가 막혔다고 표현한다. 스레드가 한 클라이언트를 위해 처리하는 동안에는 해당 스레드가 다른 클라이언트들의 요청을 처리할 수 없기 때문이다.  

이벤트 루프는 새로운 클라이언트 연결을 모니터링하고 응답 생성을 조율한다. 모든 요청들과 응답들은 이벤트 루프를 거치게 된다. 만약 이벤트 루프가 특정 지점에서 지나치게 오래 머물면 다른 클라이언트가 순서를 보장받지 못할 수 있다. 그러므로 자바스크립트 콜백은 빠른 시간 내에 완료되어야 한다. 이는 `await`나 `Promise.then` 등에서도 통용된다.  

이벤트 루프가 막히지 않는 것을 보장하는 좋은 방법은 콜백의 시간 복잡도를 유추해보는 것이다.

  
```js
// 상수 시간의 콜백
app.get('/constant-time', (req, res) => {
  res.sendStatus(200);
});

// O(n) 콜백
app.get('/countToN', (req, res) => {
  let n = req.query.n;

  for (let i = 0; i &lt; n; i++) {
    console.log(`Iter ${i}`);
  }

  res.sendStatus(200);
});

// O(n^2) 콜백
app.get('/countToN2', (req, res) => {
  let n = req.query.n;

  for (let i = 0; i &lt; n; i++) {
    for (let j = 0; j &lt; n; j++) {
      console.log(`Iter ${i}.${j}`);
    }
  }

  res.sendStatus(200);
});
```  

> 복잡한 작업의 경우 인풋을 받고나서 너무 긴 인풋의 경우 거절하는 것을 고려하는 것이 좋다. 이렇게 하면 아무리 콜백이 큰 복잡도를 가지더라도 설정한 최악의 시간 복잡도 이상은 받지 않는다는 것을 보장할 수 있다.


## 이벤트 루프 블로킹 : REDOS

#### 취약한 정규 표현식을 피하자.
이벤트 루프를 심각할 정도로 피해 입히는 것은 취약한 정규 표현식을 사용하는 것이다.  
정규 표현식은 인풋의 문자열에서 해당하는 패턴을 찾는다. 우리는 보통 한 번의 문자열 인풋은 `O(n)`의 시간이 한 번만 발생될 것이라 생각한다. 대부분의 경우 정말 한 번만 발생하지만 어떤 정규 표현식 요청은 `O(2^n)`로 이어질 수 있다. 즉, 취약한 정규 표현식은 "악의적인 인풋"에 의해 지수 시간을 소요하게 되어 REDOS로 이어지게 할 수 있다.

- `(a+)*`와 같은 중첩된 한정자를 사용하는 것을 피하자.
- `(a|a)*`와 같이 공통 부분을 가지는 절간에 OR하는 것을 피하자.
- `(a.*) \1`처럼 역참조를 피하자.
- 간단한 문자열간에 비교를 한다면 `indexOf`나 `local equivalent`를 사용하자.

#### REDOS 예시

  
```js
app.get('/redos-me', (req, res) => {
  let filePath = req.query.filePath;

  // REDOS
  if (filePath.match(/(\/.+)+$/)) {
    console.log('valid path');
  } else {
    console.log('invalid path');
  }
  
  res.sendStatus(200);
});
```  

- 위의 취약한 정규 표현식 예시는 리눅스에서 유효한 경로인지 체크하는 좋지않은 방법이다. 이는 "중첩된 한정자를 피하라"는 규칙을 위반하였으므로 위험하다.
- 만약 클라이언트가 파일경로 `///.../\n`을 요청하게 되면 이벤트 루프에서는 무한 루프에 빠지게 되면서 막히게 된다. 이러한 REDOS 공격은 대기 중인 모든 클라이언트가 정규 표현식 처리가 끝날 때까지 기다리게 한다.

#### REDOS 방어
정규 표현식이 안전한지 검사해주는 도구들이 있다. 하지만 이러한 도구가 모든 취약한 정규 표현식을 막아주는 것은 아니다.

- safe-regex
- rxxr2

다른 방법으로는 다른 정규 표현식 엔진을 사용하는 것이다.

- RE2
- node-re2

만약 URL이나 파일의 경로에 정규 표현식을 사용한다면 `regexp library`에서 예시를 찾아서 사용하거나 `ip-regex`같은 npm 모듈을 사용하자.


## 이벤트 루프 블로킹 : Node.js 코어 모듈
몇몇 노드 코어 모듈은 동기적으로 동작하는 값비싼 API를 가지고 있다.

- 암호화
  - `crypto.randomBytes`(동기적인 버전)
  - `crypto.randomFillSync`
  - `crypto.pbkdf2Sync`
- 압축
  - `zlib.inflateSync`
  - `zlib.deflateSync`
- 파일 시스템
  - 동기적인 파일 시스템 API(NFS와 같은 분산 파일 시스템에 내에 파일이 있다면 접근 시간 편차가 매우 클 수 있다.)
- 자식 프로세스
  - `child_process.spawnSync`
  - `child_process.execSync`
  - `child_process.execFileSync`

위 API들은 상당한 연산(암호화, 압축)을 필요하거나 I/O(파일 I/O)를 요구하거나 잠재적으로 둘 다 필요(자식 프로세스) 할 수 있다. 이러한 API들은 스크립트의 편의를 위해 있는 것으로 서버 사이드에서 사용하도록 의도되지 않았기 때문이다. 만약 이 API들을 이벤트 루프에서 실행한다면 이벤트 루프가 막히게 되면서 일반적인 자바스크립트 명령보다 훨씬 더 긴 시간을 소요할 것이다.

> 위 API들은 Node.js v9에 오면서 거의 완성되었다.


## 이벤트 루프 블로킹 : JSON DOS
- `JSON.parse`와 `JSON.stringify`는 잠재적으로 매우 비싼 연산이다. `O(n)`의 시간 복잡도를 가지기에 긴 시간이 소요될 수 있다. 만약 서버가 JSON 객체를 다룬다면, 특히 클라이언트에서 온 객체라면, 이벤트 루프가 다루게 될 객체의 크기를 염두해두어야 한다.
- npm 모듈이 제공하는 비동기적 JSON API인 `JSONStream`, `bfj(Big-Friendly JSON)` 를 사용하자.


## 이벤트 루프를 막지 않고 복잡한 계산하는 방법 : 파티셔닝
분할하여 계산하게 되면 분할된 모든 작업을 이벤트 루프에서 실행하지만 다른 대기 중인 이벤트에 대한 소요시간을 평균으로 맞출 수 있다. 자바스크립트에는 아래 예시처럼 현재 진행 중인 작업의 상태를 클로저에 저장할 수 있다.

  
```js
// 분할되지 않았을 때 평균, O(n) 소요
for (let i = 0; i &lt; n; i++)
  sum += i;
let avg = sum / n;
console.log('avg: ' + avg);
```  

  
```js
// 분할되었을때 평균, 비동기적으로 분할된 각각의 n은 O(1) 소요
function asyncAvg(n, avgCB) {
  // 현재 진행 중인 계산의 합을 JS 클로저에 저장.
  var sum = 0;
  function help(i, cb) {
    sum += i;
    if (i == n) {
      cb(sum);
      return;
    }

    // "비동기적 재귀".
    // 비동기적으로 다음 연산을 스케줄링.
    setImmediate(help.bind(null, i+1, cb));
  }

  // 헬퍼 함수를 시작. avgCB를 콜하는 CB.
  help(1, function(sum){
      var avg = sum/n;
      avgCB(avg);
  });
}

asyncAvg(n, function(avg){
  console.log('avg of 1-n: ' + avg);
});
```  


## 이벤트 루프를 막지 않고 복잡한 계산하는 방법 : 오프로딩
파티셔닝은 이벤트 루프만 이용하기에 여러 개의 코어를 이용하는 이점을 얻을 수 없다. **이벤트 루프는 클라이언트의 요청을 조율하는 역할이지 직접 실행하는 역할이 아니다.** 더 복잡한 작업을 하기 위해서는 이벤트 루프의 작업을 워커 풀로 옮겨야 한다.

#### 오프로딩 옵션
워커 풀에 작업을 오프로드할 것인지에 대한 옵션은 아래와 같이 2가지가 있다.

- `C++ addon`을 이용하여 설치되어 있는 Node.js의 워커 풀을 이용하는 방법. `webworker-threads` 모듈을 사용하여 자바스크립트만으로 Node.js 워커 풀에 접근할 수 있다.
- 직접 계산에 특화된 워커 풀을 생성하고 관리하는 방법. 가장 직관적인 방법은 자식 프로세스나 클러스터를 이용하는 것이다.

모든 클라이언트마다 자식 프로세스를 전부 생성하지 않는 것이 좋다. 자식을 생성하고 관리하는 것보다 클라이언트의 요청을 더 빨리 받게 되면서 서버가 포크 폭탄이 되어버릴 수 있다.

> Fork bomb(포크 폭탄)이란 프로세스가 지속적으로 자신을 복제하여 사용 가능한 시스템 리소스를 고갈시키고 이로 인해 시스템 속도를 저하시키거나 충돌시키는 것을 말한다.

#### 오프로딩 방식 단점
오직 이벤트 루프만이 애플리케이션의 네임스페이스(자바스크립트 상태)를 볼 수 있다. 즉, 이벤트 루프의 네임스페이스에 있는 자바스크립트 객체를 워커로부터 조작할 수 없다. 그래서 이를 위해서는 공유하고자 하는 객체를 직렬화, 역직렬화하여 공유해야 한다. 그 후 워커는 복사된 객체를 이용하여 연산하고 변경한 객체를 이벤트 루프에 반환한다. 이 과정에서 커뮤니케이션 비용의 형태로 오버헤드가 발생할 수 있다.


## 오프로딩과 CPU-intensive 작업, I/O-intensive 작업

#### CPU-intensive 작업
- CPU-intensive 작업은 처리를 위한 워커가 스케줄링 되었을 때만 진행되어야 하며 워커는 반드시 컴퓨터는 logical cores 중 하나에 스케줄링되어야 한다.
- 만약 4개의 논리적 코어를 가지고 있을 때 5개의 워커가 있다면 이 워커들 중 하나는 작동하지 않게 된다. 이는 해당 워커에 대한 메모리와 스케줄링에 대한 오버헤드가 발생하면서 어떤 결과도 얻지 못 하는 결과로 이어진다.

#### I/O-intensive 작업
- I/O-intensive 작업에는 외부 서비스 제공자(DNS, 파일 시스템 등)에게 요청하는 것과 이에 대한 응답을 기다리는 것이 포함된다.
- I/O-intensive 작업을 하는 워커가 응답을 기다리는 동안에 해당 워커는 아무것도 할게 없으므로 OS에 의해 스케줄링에서 제거되어 버리고 다른 워커에게 해당 요청을 넘기게 된다.
- 그러므로 I/O-intensive 작업은 관련된 스레드가 작동 중이 아니더라도 진행 중인 상태여야 한다.
- 예시로 DB나 파일 시스템과 같은 외부 서비스 제공자들은 많은 수의 대기 중인 요청을 동시에 처리하는 것에 매우 특화되어 왔다.

만약 단 하나의 워커 풀에만 의존한다면 CPU-intensive 작업과 I/O-intensive 작업 간의 서로 다른 특징으로 인해 애플리케이션 성능에 문제가 생길 수 있다. 이러한 이유로 분리된 계산 특화 워커 풀을 유지하는 것이 좋다.

> 긴 배열의 요소를 순회해야하는 것과 같은 간단한 작업은 파티셔닝이 좋은 옵션이 될 수 있다. 그러나 더 복잡한 작업은 오프로딩이 좋은 전략이다. 이벤트 루프와 워커 풀 간에 직렬화된 객체를 주고받는 데에 발생하는 오버헤드와 같은 커뮤니케이션 비용은 다수의 코어를 사용하는 이득으로 상쇄될 수 있기 때문이다. 서버가 복잡한 계산에 매우 의존적이라면 Node.js 사용하는 것 자체를 고려해봐야 한다.


## 워커 풀을 막지말자
노드는 k개의 워커로 구성된 워커 풀을 가지고 있다. 만약 오프로딩 전략을 사용 중이라면 분리된 계산 특화 워커 풀을 가지고 있을 것이다.  

각 워커는 워커 풀 큐의 다음 작업에 도달하기 위해서 현재 작업을 완료해야 한다. 클라이언트 요청을 처리해야하는 작업의 비용간에 차이가 있을 때, 어떤 작업은 빠르게 처리될 것이고, 다른 작업들은 더 오래 걸릴 것이다. 목표가 작업 소요 시간간의 편차를 최소화하는 것이라면 **작업 파티셔닝**을 이용하여 이를 달성할 수 있다.

#### 작업 소요 시간간의 편차 최소화
- 한 워커의 현재 작업이 다른 작업들보다 비싸다면 해당 워커는 다른 대기 중인 작업을 처리할 수 없다. 즉, 상대적으로 긴 작업은 해당 작업이 끝날 때까지 워커 풀의 전체 크기를 줄이게 된다.
- 한 클라이언트의 비싼 작업은 워커 풀의 처리량을 감소시키게 되고 이는 서버의 처리량 감소로 이어지게 된다. 이를 피하기 위해선 워커 풀에 위임하는 작업 길이간의 편차를 최소화하는 것이 좋다.
- 외부 시스템 접근에 대한 I/O요청(DB, 파일 시스템 등)은 블랙 박스로 다뤄지는 것이 적절하므로 이러한 I/O 요청의 상대적인 비용에 대해 인지하고 길어질 것으로 예상되는 요청은 피해야 한다.

#### 작업 파티셔닝
- 작업간의 소요 시간 편차는 워커 풀의 처리량의 감소로 이어질 수 있다.
- 작업 시간 편차를 최소화하기 위해 각 작업을 비슷한 비용이 들지만 가능한 한 작게 서브작업으로 분할해야 한다. `fs.readFile()` 을 예시로 `fs.read()`(수동 파티셔닝)가 아닌 `ReadStream`(자동 파티셔닝)을 사용해야 한다.
- 작업을 짧은 작업으로 나눴을 때 지정된 워커는 해당 작업을 끝내고 다른 작업의 서브 작업을 도울 수 있으므로 워커 풀의 전체 처리량 증가로 이어지게 된다.

#### 각기 다른 워커 풀로 라우팅
- 작업 분할의 목적은 작업 시간 편차를 최소화하기 위함이다. 짧은 작업과 긴 작업을 각기 다른 워커 풀로 라우팅시키는 것은 작업 시간 편차를 최소화하는 또 다른 방법이다.
- 이 방식의 경우, 작업 분할에 의해 발생되는 오버헤드(워커 풀 작업 생성과 워커 풀 큐를 관리하는 비용)와 워커 풀에 접근하는 비용 등을 줄일 수 있다. 이는 또한 작업을 파티셔닝할 때 생길 수 있는 실수를 방지한다.
- 이 방식의 단점은 모든 워커 풀의 워커가 공간, 시간 오버헤드를 발생시키며 CPU 시간을 경쟁적으로 사용하게 된다는 것이다. CPU-intensive 작업은 스케줄링 되었을 때만 진행되기 때문이다.


## 정리
- 노드 확장성의 핵심은 "다수의 클라이언트를 처리하는 하나의 스레드"다.
- 노드는 이벤트 루프와 워커 풀 2가지 종류의 스레드를 가진다.
- 이벤트 루프는 자바스크립트 콜백과 논블로킹 I/O에 대한 책임이 있으며 워커는 C++ 코드로 해당 작업을 실행하여 블로킹 I/O와 CPU-intensive 작업을 포함하는 비동기 요청을 완료한다. 두 종류의 스레드 모두 한번에 한 작업보다 많은 일을 하지 않는다.
- 콜백이나 작업이 긴 시간을 소요한다면 해당 스레드는 막히게 된다.
- 애플리케이션이 막히는 콜백이나 작업을 생성한다면 이는 처리량 감소로 이어지고 최악의 경우 서비스 거부 상태에 빠지게 된다.
- npm 모듈의 API를 사용하려고 한다면 이벤트 루프나 워커 풀을 막을 가능성이 있는지 체크해야 한다.


###### Reference

- https://nodejs.org/ko/docs/guides/dont-block-the-event-loop/