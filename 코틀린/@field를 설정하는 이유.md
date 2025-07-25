코틀린에서  Jackson 라이브러리의 `@JsonProperty` 사용 중 `@field:`접두사 사용이 강제 되는 이유를 프로퍼티의 **백킹 필드** 개념과 함께 정리한다.

---

### 1. 코틀린 프로퍼티 vs. 자바 필드
	
- **자바**: **필드**를 선언하고 데이터를 저장한다. 게터/세터 메서드는 필드에 접근한다.
	
- **코틀린**: **프로퍼티** 개념을 사용한다. `val` 또는 `var` 키워드로 선언하며, 내부적으로 필드, 게터, (var의 경우) 세터를 포함한다.

---

### 2. 백킹 필드란?

- 프로퍼티의 **실제 값을 저장하는 메모리 공간**이다.

- 코틀린 컴파일러가 프로퍼티의 값을 저장할 필요가 있을 때 **자동으로 생성한다**.
	
	- **생성되는 경우**: `var name: String = "Kotlin"`처럼 값을 저장하는 대부분의 프로퍼티.
	
    - **생성되지 않는 경우**: `val isAdult: Boolean get() = age >= 19`처럼 게터가 값을 직접 저장하지 않고 계산만 하는 **계산된 프로퍼티**.

---

### 3. `@field:`를 사용하는 이유

- **어노테이션 적용 대상 명시**:
    
    - 코틀린에서 주 생성자 파라미터에 어노테이션을 붙이면, 기본적으로 **파라미터 자체**에 적용된다.
        
    - 하지만 Jackson `@JsonProperty` 같은 자바 라이브러리 어노테이션은 클래스의 **필드** (즉, 백킹 필드)에 적용된다.
        
- **목적**: `@field:`를 붙여 "이 어노테이션을 프로퍼티의 **백킹 필드**에 적용해라"라고 코틀린 컴파일러에게 명시적으로 지시한다.
    
- **결과**: 이를 통해 자바 라이브러리가 코틀린 프로퍼티의 백킹 필드에 매핑할 수 있게 된다.
