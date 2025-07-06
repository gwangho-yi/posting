![[Pasted image 20250706150257.png]]

슬랙에 에러 메세지가 계속 해서 노출이 되고 있다.

무엇이 문제일까?? 아래는 에러 로그

```java
2025-06-25T14:56:43.086+09:00  INFO 1 --- [api] [io-8080-exec-56] [685b8f9b3f6057be72a949bcc69d2c32-b905b562bd62a72f] c.m.infra.web.filter.HttpLoggingFilter   : --> POST /valid/check/username
connection=Upgrade, HTTP2-Settings
host=web-api.ecs.local:8080
http2-settings=AAEAAEAAAAIAAAAAAAMAAAAAAAQBAAAAAAUAAEAAAAYABgAA
transfer-encoding=chunked
upgrade=h2c
accept=*/*
accept-encoding=gzip, deflate, br
accept-language=ko-KR,ko;q=0.9
content-type=application/json;charset=UTF-8
origin=https://site.com
referer=https://site.com/
sec-fetch-dest=empty
sec-fetch-mode=cors
sec-fetch-site=same-site
service-type=API
traceparent=00-685b8f9b3f6057be72a949bcc69d2c32-f80ff47fbf7cebd9-01
user-agent=Mozilla/5.0 (Linux; Android 10; WebView) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.104 Mobile Safari/537.36
x-amzn-trace-id=Root=1-685b8f9b-6378952d442c952e6bb289c6
cookie=pass_req=igwkvmdqvjvnovwjjsftrxolwrtebh
{
    "username": "lsr0217"
}
 -->
```

해당 에러는 `Username` 객체의 역직렬화 과정에서 조건을 충족하지 못해 발생한 에러다.

```java
public class Username implements Serializable {

    @Nonnull
    public final String value;

    Username(
        @Nonnull final String value
    ) {
        if (value.isBlank()) {
            throw new BadArgumentException("common.username.required");
        }

        if (validateLength(value)) {
            throw new BadArgumentException("common.username.miss-pattern-character");
        }
    }   
}
```

해당 객체 생성 중에 `BadArgumentException` 에러가 발생할 때는 슬랙 에러 메세지 보내기 기능을 설정하지 않았는데 왜 이런 문제가 발생하는 걸까??

`GlobalExceptionHandler` 에서 지정한 `BadArgumentException` 에러 처리 메서드는 로그 표시와 에러 메세지 리턴 정도만 구현했으나 이 메서드가 처리 되지 않고 `HttpMessageNotReadableException` 에러에 대한 에러가 우선 처리되는 현상 발생했다. 여기서 슬랙 메세지가 전송됐다.

```java

public class GlobalExceptionHandler {

	@ExceptionHandler(BadArgumentException.class)
    public ServerResponse<Object> handleBadArgumentException() {
        log.error(
            "에러 메세지~~"
        );

        return ServerResponse.builder()
            .result(ServerResult.from(e))
            .build();
    }
    
    
    @ExceptionHandler(HttpMessageNotReadableException.class)
    public ServerResponse<Object> handleHttpMessageNotReadableException() {
        Throwable cause = e.getCause().getCause();

        log.error(
            "에러 메세지~~"
        );

        sendSlack.execute((Exception) cause);

        return ServerResponse.badRequest(e.getCause().getCause().getMessage());
    }
}
```

원인이 뭘까 ?? 분명히 난 `BadArgumentException`을 발생 시키도록 했는데 말이다.

디버그 모드로 출력해보니 객체 변환 실패 시점에는 `BadArgumentException` 이 발생하는게 맞지만, 최종적으로는 `HttpMessageNotReadableException` 으로 throw되고 있었다.

## HttpMessageNotReadableException

- 스프링에서 제공하는 예외 클래스
- HTTP 요청을 읽을 수 없을 때 발생
- JSON 요청 본문 파싱 실패 시 발생

흐름은 이렇다.

1. Json → Java 객체 변환 시도
2. 생성자에서 `BadArgumentException` 발생
3. Jackson이 `JsonMappingException` 로 매핑
    - Jackson이 JSON 변환 중 발생시키는 예외
4. Spring에서 `HttpMessageNotReadableException` 로 매핑
    - Spring이 HTTP 메시지 읽기 실패 시 발생시키는 예외
    - 최종적으로는 http requet body 의 내용을 역직렬화 불가능 하기 때문에 발생.

최종적으로는 4번 이유 때문에 `HttpMessageNotReadableException` 에러가 발생하게 된다**`HttpMessageConverter`** 를 상속받아서 중간에 가로채는 방법도 있지만, 현재 상황에서는 slack 메세지만 보내지만 않으면 되기 때문에 `instaceof` 로 해결 가능하다.

```java

public class GlobalExceptionHandler {

	@ExceptionHandler(BadArgumentException.class)
    public ServerResponse<Object> handleBadArgumentException() {
        log.error(
            "에러 메세지~~"
        );

        return ServerResponse.builder()
            .result(ServerResult.from(e))
            .build();
    }
    
    
    @ExceptionHandler(HttpMessageNotReadableException.class)
    public ServerResponse<Object> handleHttpMessageNotReadableException() {
        Throwable cause = e.getCause().getCause();

        log.error(
            "에러 메세지~~"
        );

        if (cause instanceof BadArgumentException) {
            return ServerResponse.badRequest(e.getCause().getCause().getMessage());
        }
        
        sendSlack.execute((Exception) cause);

        return ServerResponse.badRequest(e.getCause().getCause().getMessage());
    }
}
```