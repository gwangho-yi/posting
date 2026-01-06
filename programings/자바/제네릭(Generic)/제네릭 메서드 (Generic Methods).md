
## 제네릭 메서드  (Generic Methods)
메서드 본인이 사용할 타입을 도입부에 적어놓은 메서드를 말한다. 쉽게 말해서 메서드의 반환타입 앞에 타입 파라메터를 적어놓은 메서드를 말한다.
```java
public static <P> void print(P p) {  
    System.out.println(p);  
}
```
제네릭 타입과 비슷하지만 타입 매개변수의 스코프는 메서드 범위에 제한된다.

클래스와 독립적인 static 메서드는 클래스와는 독립적이기 때문에 메서드 자신이 사용할 타입 파라메터를 별도로 선언해줘야한다.

```java
public class Util {  
  
    public static <K, V> boolean compare(Pair<K, V> p1, Pair<K, V> p2) {  
        return p1.getKey().equals(p2.getKey())  
            && p1.getValue().equals(p2.getValue());  
    }  
}  
  
public class Pair<K, V> {  
  
    private K key;  
    private V value;  
  
    public Pair(K key, V value) {  
        this.key = key;  
        this.value = value;  
    }  
    public void setKey(K key) { this.key = key; }  
    public void setValue(V value) { this.value = value; }  
    public K getKey()   { return key; }  
    public V getValue() { return value; }  
}
```

> [!NOTE] 네이밍 컨벤션
>  오라클 document에는 제네릭 네이밍 컨벤션을 소개한다.
> - E - Element (used extensively by the Java Collections Framework)
> - K - Key
> - N - Number
> - T - Type
> - V - Value
> - S,U,V etc. - 2nd, 3rd, 4th types
