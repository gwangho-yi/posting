물음표(?)는 와일드 카드라고 불린다. 와일드 카드는 파라메터, 필드, 지역변수, 반환 타입 같은 여러가지 상황에서 사용할 수 있다. 와일드 카드를 사용하지 않는 상황은 다음과 같다.
인자 - 제네릭 메서드 실행을 위한,
제네릭 클래스 인스턴스 생성,
또는 상위 타입

`? extends Type` 이 문법인데

## 와일드 카드와 상한 제한 파라메터의 차이
와일드 카드(wild card)와 상한 제한 파라메터는 둘 다 `extends` 키워드를 사용한다. 이때, 어떤 때에 두 가지를 사용하면 좋을지 공통점과 차이점을 간단하게 살펴보자.

### 공통점
- 하위 타입들을 유연하게 사용하고 싶다.

### 차이점
- 와일드 카드
	- read만 가능, write 불가
	- 타입 반환은 ? 로 불가능하다.
- 상한 제한 파라메터
	- T 타입 반환 가능하다.

```java

// Wildcard: 유연하게 받지만 읽기만  
public void process(List<? extends Animal> animals) {  
    for (Animal a : animals) {  
        a.eat(); // ✅ 읽기  
    }  
    // animals.add(new Dog()); // ❌ 쓰기 불가  
}  
  
// Type Parameter: 유연하게 받고 쓰기도 가능  
public <T extends Animal> void process(List<T> animals, T newAnimal) {  
    for (T a : animals) {  
        a.eat(); // ✅ 읽기  
    }  
    animals.add(newAnimal); // ✅ 쓰기도 가능  
}

```
