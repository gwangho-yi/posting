동기/비동기 그리고 블로킹/논블로킹은 처음에 비슷한 말처럼 느껴져서 자꾸 뒤섞였다. 개념 자체가 비슷한 게 아니라 보는 축이 다른 건데, 그걸 이해하고 나서야 명확해졌다.

핵심만 먼저:

동기/비동기 : 결과 처리가 내 호출 흐름과 동일한가, 아니면 내 흐름과 분리되어 결과 처리를 통보 받는가
블로킹/논블로킹 : 호출 후 제어권이 호출자인 나에게 있는가, 아니면 호출된 자가 제어권을 가지고 있는가

---

## 동기 Synchronous

결과가 반환될 때까지 호출자가 직접 기다리는 방식이다. 결과 처리가 내 호출 흐름 안에서 순서대로 이루어진다.

```java
String result = callSomeMethod(); // callSomeMethod()가 끝날 때까지 여기서 대기
System.out.println(result);       // result가 반환된 후에야 실행
```

카운터 앞에 서서 음료가 나올 때까지 기다렸다가 직접 받아가는 것과 같다. 순서가 보장된다는 장점이 있다.

## 비동기 Asynchronous

호출 후 결과를 기다리지 않고 다음 작업을 이어서 하는 방식이다. 결과가 준비되면 콜백이나 이벤트로 통보받는다.

```java
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    return callSomeMethod(); // 별도 스레드에서 실행
});

System.out.println("callSomeMethod가 끝나기 전에 이미 여기가 실행됨");

future.thenAccept(result -> {
    System.out.println(result); // 완료 후 콜백으로 처리
});
```

진동벨을 받고 자리에 앉아서 다른 일 하다가 벨이 울리면 가서 음료를 받는 것이다. 결과 처리가 내 흐름과 분리되어 있다.

## 블로킹 Blocking

호출된 함수가 제어권을 가지고 있어서, 호출한 쪽이 결과를 받을 때까지 멈추는 방식이다.

```java
InputStream is = socket.getInputStream();
byte[] buffer = new byte[1024];
int bytesRead = is.read(buffer); // 데이터가 올 때까지 스레드가 여기서 멈춤
System.out.println("읽은 바이트: " + bytesRead);
```

`is.read(buffer)`를 호출하는 순간, 데이터가 실제로 올 때까지 스레드가 블록된다. 제어권이 `read()` 내부에 있기 때문이다.

## 논블로킹 Non-Blocking

호출 후 즉시 제어권이 호출자에게 돌아오는 방식이다. 결과가 아직 준비되지 않았으면 그 사실을 바로 반환한다.

```java
SocketChannel channel = SocketChannel.open();
channel.configureBlocking(false); // 논블로킹 모드

ByteBuffer buffer = ByteBuffer.allocate(1024);
int bytesRead = channel.read(buffer); // 데이터 없으면 즉시 0 반환, 스레드 멈추지 않음

if (bytesRead == 0) {
    // 아직 데이터 없음 - 다른 작업 처리 가능
}
```

결과가 없어도 즉시 반환하기 때문에 제어권이 호출자에게 있다.

---

## 4가지 조합

동기/비동기와 블로킹/논블로킹은 독립적인 개념이라 2x2 조합이 가능하다.

| | 블로킹 | 논블로킹 |
|---|---|---|
| **동기** | 일반적인 메서드 호출 | 폴링 (결과 올 때까지 직접 반복 확인) |
| **비동기** | `Future.get()` | Node.js, CompletableFuture, 코루틴 |

### 동기 + 블로킹

가장 일반적인 방식이다. 호출 → 대기 → 결과 처리가 순서대로 이루어진다. 우리가 평소에 쓰는 대부분의 메서드 호출이 이 방식이다.

### 동기 + 논블로킹

결과가 나올 때까지 호출자가 직접 반복해서 확인(폴링)하는 방식이다. 제어권은 호출자에게 있지만, 결과를 스스로 계속 확인해야 한다.

```java
while (true) {
    int bytesRead = channel.read(buffer); // 논블로킹 - 즉시 반환
    if (bytesRead > 0) break;             // 결과가 생기면 탈출
    // 결과 확인 사이에 다른 작업 처리 가능
}
```

### 비동기 + 블로킹

드문 경우지만, `Future.get()`처럼 비동기로 작업을 시작했지만 결과를 기다리면서 블록되는 패턴이다.

```java
Future<String> future = executor.submit(() -> callSomeMethod()); // 비동기 호출
String result = future.get(); // 여기서 블로킹 - 결과 나올 때까지 멈춤
```

비동기로 시작했는데 결과를 동기적으로 기다리는 셈이라, 비동기의 이점을 제대로 살리지 못한다.

### 비동기 + 논블로킹

가장 효율적인 방식이다. 호출 후 즉시 제어권이 돌아오고, 결과는 콜백/이벤트로 처리된다. Node.js의 이벤트 루프, Java의 `CompletableFuture`, Kotlin의 코루틴이 이 방식이다.

---

## 정리

| 구분 | 관심사 | 핵심 |
|---|---|---|
| 동기 Synchronous | 결과 처리 시점 | 결과를 내 흐름 안에서 직접 처리 |
| 비동기 Asynchronous | 결과 처리 시점 | 결과를 콜백/이벤트로 통보받아 처리 |
| 블로킹 Blocking | 제어권 | 호출 후 제어권이 호출된 쪽에 있음 |
| 논블로킹 Non-Blocking | 제어권 | 호출 후 제어권이 즉시 나에게 돌아옴 |

처음에는 동기 = 블로킹, 비동기 = 논블로킹이라고 생각했는데 사실 전혀 다른 축의 개념이었다. 결과 처리 방식과 제어권의 위치를 분리해서 생각하면 4가지 조합이 자연스럽게 이해된다.
