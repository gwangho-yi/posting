
때때로 매개변수화 된 타입의 인자를 제한하고 싶은 때가 있을 것이다. 예를 들어, 숫자를 사용하는 메서드가 있다면 `Number`나 하위 클래스들로 범위를 제한하는 것이다. 이때 제한된 타입 매개변수를 사용할 수 있다.

문법은 타입 매개 변수의 이름, `extends` 키워드, 상한 타입을 적는다.
```java
public class Box<T> {  
  
    private T t;  
  
    public void set(T object) {  
        this.t = object;  
    }  
  
    public <U extends Number> void inspect(U u) {  // Number를 상한 타입으로 제한
        System.out.println("T: " + t.getClass().getName());  
        System.out.println("U: " + u.getClass().getName());  
    }  
  
    public T get() {  
        return t;  
    }  
}

public static void main(String[] args) {  
    Box<Integer> box = new Box<>();  
      
    box.set(10);  
      
    box.inspect("10"); // 에러  
}
```


또한, 제네릭 타입에서 제한된 타입을 파라메터를 사용한다면, 타입의 경계를 제한했기 때문에 이 제한된 타입이 가진 메서드를 호출할 수 있게 된다.

```java
public class NaturalNumber<T extends Integer>{  
  
    private final T n;  
  
    public NaturalNumber(T n) {  
        this.n = n;  
    }  
  
    public boolean isEven() {  
        return n.intValue() % 2 == 0; 
    }  
}

public static void main(String[] args) {  
  
    NaturalNumber<Number> naturalNumber = new NaturalNumber<>(10.1);  
    boolean isEven = naturalNumber.isEven();  
    System.out.println("isEven = " + isEven);  
}
```

### 다중 제한 (Multi Bounds)
타입 매개변수는 다중 제한을 걸 수  있다. 이때 `T`는 다중 제한들의 하위 타입이어야하며, 클래스와 인터페이스들을 상위 타입으로 나열한다면 반드시 클래스가 처음에 위치해야한다.
```java
class A{}  
interface B{}  
interface C{}

class D <T extends A & B & C>{}
```
`T`는 `A`, `B`, `C`의 하위 타입이어야하며, `A`는 클래스기 때문에  다중 타입 나열의 가장 첫번째에 위치해야한다.

아래와 같이 다중 제한을 지정했을 때, T는 Number와 Comparable을 둘 다 만족해야한다.
```java
<T extends Number & Comparable<T>>
```


### 제네릭 메서드와 제한된 타입 파라메터
제한된 타입 파라메터는 제네릭 알고리즘 구현의 핵심이다. 아래 코드는 `T[]`의 요소들이 elem보다 클 때 숫자를 카운팅한다.
```java
public static <T> int countGreaterThan(T[] array, T elem) {  
    int count = 0;  
    for (T item : array) {  
        if (item > elem) { // 컴파일 에러  
            count++;  
        }  
    }  
    return count;  
}
```
비교연산자 `>`는 사실상 원시 타입 int, long, double, char 등을 비교 연산한다.  사실상 타입 자체는 비교를 수행할 수 없는 타입들이 지정될 수 있기 때문에 컴파일러 입장에서는 당연히 에러를 띄울 것이다. 여기서 우리는 제한된 타입 파라메터를 고려할 수 있다

```java
public static <T extends Comparable<T>> int countGreaterThan(T[] array, T elem) {  
    int count = 0;  
    for (T item : array) {  
        if (item.compareTo(elem) > 0) {  
            count++;  
        }  
    }  
    return count;  
}
```

`Comparable`를 상속하는 타입들, `Interger` 와 같은 숫자 타입 등 비교 가능한 모든 타입들은 `compareTo`로 비교 연산 가능하다. 