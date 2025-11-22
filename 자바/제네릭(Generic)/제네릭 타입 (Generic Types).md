
## 제네릭 타입 (Generic Types)

제네릭 타입은 타입이 매개변수화된 제네릭 클래스나 제네릭 인터페이스를 말한다.

제네릭 타입 선언은 아래처럼 할 수 있다. 꺾쇠 괄호 안에 타입을 지정한다. 하나만 지정해도 되고 연속적으로 지정해도 된다.
```java
class name<T1, T2, ..., Tn> { /* ... */ }
```

단순하게 Object만 선언된 Box 클래스를 제네릭 타입으로 변경해보자.
```java
public class Box {  // 일반 버전
  
    private Object object;  
  
    public void set(Object object) {  
        this.object = object;  
    }  
  
    public Object get() {  
        return object;  
    }  
  
}
```

```java

public class Box<T> { // 제네릭 버전

    private T t;

    public void set(T t) { this.t = t; }
    public T get() { return t; }
}
```

`Box<T>`를 참조하려면 T를 구체적인 값으로 변경하여 제네릭 타입 호출을 실행해야한다.
`T` 에 `Inteager` 를 인자로 전달하면 된다. String 이나 다른 타입을 지정해도 무관하다.
이제 이렇게 선언된 제네릭 타입은 String 값을 제외한 타입의 값은 설정할 수 없게 된다.
```java
Box<Integer> integerBox;
```


T를 선언하면 클래스 자체에 어떤 효과를 발휘하는건 아니고 필드와 메서드의 매개변수들의 타입을 T로 지정할 수 있게 된다. 즉, 타입 자체를 매개변수화할 수 있게 된다. 
일반적으로 매개변수라고 하면 값이나 함수를 값으로 받아서 사용하지만 제네릭 타입은 타입을 매개변수로 받아서 클래스가 가진 필드나 메서드들의 타입을 지정할 수 있게 된다. 
```java
Box<Integer> box = new Box<>();  
box.set(10);  
box.set("string"); // 컴파일 에러
```

> [!NOTE] 다이아몬드 연산자 `<>`
> 그리고 자바 7버전 이상부터는 선언된 컴파일러가 문맥상 타입의 인자를 판단하거나 추론할 수 있는 경우, 제네릭 클래스의 생성자 호출에 필요한 인자를 `<>` 다이아몬드로 대체 할 수 있다. 
쉽게 말해 왼쪽에 타입이 있다면 오른쪽에는 타입을 생략해도 된다.


그렇다면 static 메서드는 어떨까? 
사실상 클래스와는 독립적인 static 메서드는 클래스의 타입 파라메터에 영향을 받을까?