# What is Jackson?

jackson 라이브러리는 흔히 “자바 JSON 라이브러리” 혹은 “자바에서 사용하는 JSON 파서” 으로 알고 있다. 근데 찾아보니깐 단순히 JSON과 관련된 동작만이 아닌 CSV, TOML, XML, YAML 에도 적용된다. 단순히 JSON과 관련된 기능들만이 아닌 여러 기능을 가진 라이브러리 프로젝트들이 존재하기 때문이다.

Jackson에는 3가지 핵심 라이브러리(`core`, `databind`, `annotations`) 그리고 데이터 포맷 라이브러리, 데이터 타입 라이브러리 등 다른 보조 라이브러리들이 집합되어 있다. 사실상 Jackson 라이브러리가 아니라 Jackson은 프로젝트다. `spring-boot-starter-web`에 기본적으로 포함 되어 있기 때문에 사실 평소에 눈여겨 볼 일은 없다.

# 라이브러리 특징 파악

## jackson-core

모든 jackson 라이브러리들의 공통 동작을 관리한다. annotations와 databind는 core 라이브러리에 모두 의존적이다. 객체 -> Json , Json -> 객체 모든 매핑에 관련된 동작을 수행하는건 사실 이 core 라이브러리가 있기 때문에 가능한 것이다. `JsonToken` , `JsonParser`, `JsonFactory`, `JsonGenerator` 등이 주요 클래스들이다.
## jackson-databind