
## 정의

클래스, 인터페이스, 메서드를 정의할 때, 타입(클래스, 인터페이스)을 파라메터로 선언하게 해주는 타입 추상화다. 제네릭을 사용함으로써 얻는 이점은 아래와 같다.

 1. 캐스팅이 불필요하다.
	제네릭 없던 시절에는 저장된 값의 타입을 개발자가 유추해서 타입을 지정해야했다.
	문자열은 String으로 직접 캐스팅을 해줘야했다.
	```java
	List list = new ArrayList();
	list.add("hello");
	String s = (String) list.get(0);
	```
	제네릭을 사용하면 직접 캐스팅을 하지 않아도 된다. 이미 선언 시점에 타입을 지정할 수 있기 때문이다.
	```java
	List<String> list = new ArrayList<String>();
	list.add("hello");
	String s = list.get(0);
	```

2. 컴파일 시점에 타입 검사가 가능하다.
	 캐스팅 코드는 런타임에 실행된다. 잘못 캐스팅된 코드가 있다면 런타임에 오류가 발생할 수 있다.
	 ```java
	List list = new ArrayList();  
	list.add("hello");  
	String s = (String) list.get(0);  
	Integer a = (Integer) list.get(0); // 런타임 시 오류 발생
	 ```
	제네릭을 사용하면 컴파일 시점에 이미 에러가 발생한다. 런타임에 발생하는 오류를 사전에 방지할 수 있다.
	 ```java
	List<String> list = new ArrayList<String>();  
	list.add("hello");  
	String s = (String) list.get(0);  
	Integer a = (Integer) list.get(0); // 컴파일 시 오류 발생
	 ```


## 제네릭 타입

제네릭 타입은 타입이 매개변수화된 제네릭 클래스나 제네릭 인터페이스를 말한다.

제네릭 타입 선언은 아래처럼 할 수 있다. 꺾쇠 괄호 안에 타입을 지정한다. 하나만 지정해도 되고 연속적으로 지정해도 된다.
```java
class name<T1, T2, ..., Tn> { /* ... */ }
```

단순하게 Object만 선언된 Box 클래스를 제네릭 타입으로 변경해보겠다.
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

> 오라클 document에는 제네릭 네이밍 컨벤션을 소개한다.
>  - E - Element (used extensively by the Java Collections Framework)
> - K - Key
> - N - Number
> - T - Type
> - V - Value
> - S,U,V etc. - 2nd, 3rd, 4th types

`Box<T>`를 참조하려면 T를 구체적인 값으로 변경하여 제네릭 타입 호출을 실행해야한다.
`T` 에 `Inteager` 를 인자로 전달하면 된다. String 이나 다른 타입을 지정해도 무관하다.
이제 이렇게 선언된 제네릭 타입은 String 값을 타입의 값은 설정 할 수 없게 된다.
```java
Box<Integer> integerBox;
```

### 다이아몬드 연산자 

그리고 자바 7버전 이상부터는 선언된 컴파일러가 문맥상 타입의 인자를 판단하거나 추론할 수 있는 경우, 제네릭 클래스의 생성자 호출에 필요한 인자를 `<>` 다이아몬드로 대체 할 수 있다. 
쉽게 말해 왼쪽에 타입이 있다면 오른쪽에는 타입을 생략해도 된다.```mermaid
class BankAccount
    BankAccount : +String owner
    BankAccount : +Bigdecimal balance
    BankAccount : +deposit(amount)
    BankAccount : +withdrawal(amount)
```


```java
Box<Integer> integerBox = new Box<>();
```


https://docs.oracle.com/javase/tutorial/java/generics/types.html