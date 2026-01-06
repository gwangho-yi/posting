
와일드 카드는(?) 알려지지 않은 타입(unknown type)을 나타낸다.
제네릭 타입 파라메터는 공변(Covariance)을 허용하지 않으며 불변(Invariance)이다. 불변이기 때문에 Type Safe 하지만 변수의 유연한 사용은 불가능하다. 이때, 와일드 카드를 사용해 볼 수 있다. 아래 예시를 보자.

```java
public class Parent {}

public class Child extends Parent {}

void invoke(List<Parent> list) {}

List<Child> childList = new ArrayList<>();  
  
List<Parent> parentList = new ArrayList<>();

invoke(parentList) // 호출 가능

invoke(childList) // 호출 불가능. 컴파일 에러
```

`invoke` 메서드의 매개 변수는 `List<Parent>`이기 때문에 `List<Child>`를 인자 값으로 호출 불가능하다. 바로 이때 와일드 카드를 사용 가능하다.

```java
void invoke(List<? extends Parent> list) {}

invoke(childList) // 호출 가능
```

처음 말했다시피 제네릭 타입 파라메터는 불변이며, `List<Child>`는 `List<Parent>`의 하위 타입이 아니다. 즉, 공변이 허용되지 않는다. 하지만 와일드카드를 사용함으로써 `Parent`와 상속 관계에 있는 타입들은 어떤 것이든 허용하게 되었다. 다시 말해, 공변을 허용하게 되어 좀 더 유연하게 변수를 사용할 수 있게 되었다.

와일드 카드에서 배우는 것들은 아래와 같다.
1. 상한 제한 와일드 카드 (Upper Bounded Wildcards)
2. 제한 없는 와일드 카드 (Unbounded Wildcards)
3. 하한 제한 와일드 카드 (Lower Bounded Wildcards)
4. 와일드 카드 캡쳐 (Wildcard Capture)


## 상한 제한 와일드 카드(Upper Bounded WildCards)
상한 제한 와일드 카드를 사용하면 변수의 제한 사항을 완화할 수 있다.
와일드카드(?), `extends`, 상한 제한 순의로 정의한다. 
List를 예로들자면, `List<? extends Number> list` 가 상한 제한 와일드카드 문법이다. Number와 Number의 하위 타입들을 지정할 수 있다.
클래스의 상한은 분명히 알고 있고, 하한은 알 수 없을 때 사용한다.
### 특징 
상한 제한 와일드카드는 읽기는 가능하지만 쓰기는 불가능하다는 특징이 있다. 아래 예시를 보자. 
```java 
List<? extends Parent> parents = new ArrayList()<>; 

// 읽기 가능 - Parent 타입으로 안전하게 읽을 수 있음 
Parent parent = parents.get(0); 

// 쓰기 불가 - 컴파일 에러 
parents.add(new Child());
parents.add(new Parent()); 
```
왜 쓰기가 불가능할까? `List<? extends Parent>`는 실제로 `List<Child>`, `List<Parent>` 등 무엇이든 될 수 있다. 만약 실제 타입이 `List<Child>`인데 `Parent`를 넣는 것을 허용한다면, 나중에 `Child` 타입으로 읽을 때 타입 불일치 문제가 발생한다. 이는 제네릭이 해결하려던 런타임 타입 에러 문제를 다시 만들게 되므로, 컴파일러는 쓰기를 막는다.

풀어서 설명하자면 아래 예시와 같다. 실제로는 절대 동작될 수 없는 코드이며 어디까지나 예시다.
상한 제한 와일드 카드에 쓰기가 가능했다고 가정해보자.
```java
List<? extends Parent> parents = new ArrayList()<>;

// 쓰기
parents.add(new Child());
parents.add(new Parent());

// 읽기
(Child) parents.get(0);
(Parent) parents.get(1);
```
쓰기가 가능하더라도 읽어오는 시점에 Child는 Parent가 아니기 때문에 타입을 읽어올 수 없다. 그렇다고 캐스팅을 실시할까? 그렇다면 제네릭의 등장 이전의 문제를 반복하는 꼴이다. 제네릭이 해결하고자 했던 **타입 안정성**과 **명시적 캐스팅 제거** 이 두 가지를 반하게 된다. 이러한 이유로 인해서 상한 제한 와일드 카드는 쓰기가 불가능하다.


## 제한 없는 와일드카드(Unbounded Wildcards) 
제한 없는 와일드카드는 `<?>`로 표현하며, 모든 타입을 허용한다. 타입 정보가 전혀 필요 없을 때 사용한다. 아래 예시를 보자. 
```java 
// 타입 상관없이 리스트 크기만 확인 
public int getSize(List<?> list) { return list.size(); } 

// 사용 - 모든 리스트 타입 가능 
getSize(new ArrayList()); 
``` 
### 특징 
제한 없는 와일드카드도 쓰기는 불가능하다. null만 추가 가능하다.
```java
public void test(List<?> list) { 
	// 읽기 - Object로만 가능
	Object obj = list.get(0); 

	// 쓰기 - 불가능 
	// list.add(new Parent());
	// list.add(new Child()); 
	
	// null만 가능 
	list.add(null); 
	
	// 삭제는 가능 
	list.remove(0); 
	list.clear(); 
}

``` 
