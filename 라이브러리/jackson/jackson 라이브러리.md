# What is Jackson?

jackson 라이브러리는 흔히 “자바 JSON 라이브러리” 혹은 “자바에서 사용하는 JSON 파서” 으로 알고 있다. 근데 찾아보니깐 단순히 JSON과 관련된 동작만이 아닌 CSV, TOML, XML, YAML 에도 적용된다. 단순히 JSON과 관련된 기능들만이 아닌 여러 기능을 가진 라이브러리 프로젝트들이 존재하기 때문이다.

Jackson에는 3가지 핵심 라이브러리(`core`, `databind`, `annotations`) 그리고 데이터 포맷 라이브러리, 데이터 타입 라이브러리 등 다른 보조 라이브러리들이 집합되어 있다. 사실상 Jackson 라이브러리가 아니라 Jackson은 프로젝트다. `spring-boot-starter-web`에 기본적으로 포함 되어 있기 때문에 사실 평소에 눈여겨 볼 일은 없다.

# 라이브러리 특징 파악

## jackson-core

모든 jackson 라이브러리들의 공통 동작을 관리한다. `annotations`와 달리 `databind`는 core 라이브러리에 의존적이다. 객체 -> JSON , JSON -> 객체 모든 매핑에 관련된 동작을 수행하는건 사실 이 core 라이브러리가 있기 때문에 가능한 것이다. 주요 클래스로는 `JsonToken`, `JsonFactory`,  `JsonParser`, `JsonGenerator` 등이 있다.

### JsonToken
Enum 클래스로 Json의 컨텐츠를 만드는 문자열 모음이라고 할 수 있겠다. `{`,`}`, `[`, `]` 등의 문자들이 열거형으로 관리된다. 필드명 그리고 필드의 값이 문자형인지 숫자형인지 boolean 값인지를 구분할 수 있는 이유가 `JsonToken`이 있기 때문이다.
```java
// Object의 시작 값을 알리는 { 를 반환
START_OBJECT("{", JsonTokenId.ID_START_OBJECT),  
  
// Object의 시작 끝을 알리는 } 를 반환
END_OBJECT("}", JsonTokenId.ID_END_OBJECT),  

// 배열 값의 시작을 알리는 [ 반환
START_ARRAY("[", JsonTokenId.ID_START_ARRAY),  
  
// 배열 값의 끝을 알리는 ] 반환
END_ARRAY("]", JsonTokenId.ID_END_ARRAY)


// 이 밖에도 true , false 값 등이 있음
```

### JsonFactory
`JsonGenerator`와 `JsonParser`를 생성하는 팩토리 클래스이고, jackson core의 진입점이다. `ObjectMapper`가 내부적으로 `JsonFactory`를 사용한다. `JsonFactoryBuilder` 를 사용하여 builder 패턴으로도 사용할 수 있다. 2.10 버전 이후부터는 builder 방식을 더 권고한다.

```kotlin

val factory = JsonFactory()

// builder 방식
val factory = JsonFactoryBuilder().build()

// parser 생성
val parser = factory.createParser("""{"name":"Jack","age":20}""")

// generator 생성
val generator = factory.createGenerator(ByteArrayOutputStream(), JsonEncoding.UTF8)

```


### JsonGenerator
JSON을 만드는 구현체다. 
```kotlin
val factory = JsonFactoryBuilder().build()  
  
val output = ByteArrayOutputStream()  
  
// generator 생성  
val generator = factory.createGenerator(output, JsonEncoding.UTF8)  
  
generator.writeStartObject()  // {  
generator.writeStringField("name", "Gwangho")  // "name": "Gwangho"  
generator.writeNumberField("age", 30)       // "age": 30  
generator.writeEndObject()    // }  
  
generator.close()  
  
println(output.toString()) // {"name":"Gwangho","age":30}
```

### JsonParser
JSON을 스트리밍 방식으로 읽을 때 사용하는 구현체다.
```kotlin
val parser = factory.createParser("""{"name":"Gwangho","age":30}""")  
  
while (parser.nextToken() != null) {  
    when (parser.currentToken) {  
        JsonToken.FIELD_NAME -> println("필드: ${parser.currentName}")  
        JsonToken.VALUE_STRING -> println("문자열: ${parser.text}")  
        JsonToken.VALUE_NUMBER_INT -> println("숫자: ${parser.intValue}")  
        else -> {}  
    }  
}

/*
필드: name
문자열: Gwangho
필드: age
숫자: 30
*/
```
 
 `ObjectMapper` 가 내부적으로 `JsonFactory`를 사용하여 JSON 변환 처리를 한다. 이 때, `JsonGenerator`, `JsonParser` 가 호출 된다. 그렇기 때문에 사실 실제로 두 가지를 직접 만지는 경우는 드물다. `ObjectMapper`도 사실 단위 테스트 중에 종종 사용하는 정도인데, 위 두 가지와 차이점이라면 `ObjectMapper`는 JSON을 읽어 객체를 변환하고 객체는 메모리에 올라가지만 `JsonParser`는 단순히 JSON 텍스트를 읽어서 토큰 정보를 읽는 정도다. `JsonParser`를 사용할 경우에는 개발자가 직접 객체를 만들어 줘야한다. core api는 저수준 스트리밍 api 라고도 소개되어 있는데, 대용량 JSON을 메모리 문제로 한 줄 한 줄 처리해야하는 레벨이 아니라면 두 가지는 사실 만날 일이 많지는 않다. 

## jackson-annotaition
JSON 직렬화/역직렬화를 제어하는 어노테이션 세트로 스프링 개발할 때, 사실 너무나 많이 사용하기 때문에 눈에 대부분은 눈에 익을 것이다. 자주 사용하는 어노테이션들은 아래와 같다.

```kotlin
@JsonProperty : JSON의 필드명 변경
@JsonIgnore : JSON 변환에서 제외
@JsonIgnoreProperties : 알 수 없는 필드 무시
@JsonIgnore : JSON 변환에서 제외
@JsonIgnoreProperties: 알 수 없는 필드 무시
@JsonValue : 직렬화 시 해당 메서드 결과 사용
@JsonCreator : 직렬화 시 사용할 생성자
@JsonTypeInfo / @JsonSubTypes : 타입 처리
@JsonInclude: null 필드 제외

```

## jackson-databind
