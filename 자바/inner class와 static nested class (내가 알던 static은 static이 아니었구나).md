중첩 클래스 사용 중에 생각해보니 static 클래스는 왜 다른 스레드와 공유가 발생하지 않는 것인가 의문을 해결하는 김에 중첩 클래스에 대한 정리도 한꺼번에 하고자 한다.

자바에서의 중첩 클래스 Nested Class
- 정적 중첩 클래스 static nested class
- 비정적 중첩 클래스 inner class  

## inner class는 주로 어떤 때에 사용하는가?  

외부 클래스와 내부 클래스가 강결합 된 구조에서 사용한다. 풀어서 얘기하면 아래와 같다.  
- 부가적인 기능을 가진 캡슐화 된 서브 클래스가 필요할 때.  
- 내부에서 외부의 인스턴스 변수에 접근하는 구조일 때.  
- 외부 클래스와 내부 클래스가 생명주기를 함께해야 할 때.  
  
```java  
/**  
    inner class*/  
class ShoppingCart {  
    private List<Item> items;    
    private double discountRate = 0.5;  
    
    // 요약 정보를 가진 서브 클래스  
    class CartSummary {  
        public double getSubtotal() {            
	        return items.stream() // 외부 클래스의 인스턴스 접근  
		                .mapToDouble(Item::getPrice)                
		                .sum();        
		}  
		
        public double getTotalDiscount() {            
			return getSubtotal() * discountRate;  // 외부 변수 사용  
        }    
	}
}  
  
  
ShoppingCart cart1 = new ShoppingCart();  
cart1.addItem(new Item("노트북", 1000));  
cart1.addItem(new Item("마우스", 30));  
  
ShoppingCart.CartSummary summary = cart1.getSummary();  
System.out.println(summary.getDiscount());  // cart1의 할인 금액 표시  
```  


### inner 클래스의 메모리 누수 문제  

내부 클래스를 사용함으로써, 메모리 누수가 일어나는 경우는 크게 2가지다.  
1. 외부 클래스가 참조하는 인스턴스의 크기가 클 경우  
2. 외부 클래스의 갯수가 많은 경우  

```java  
/**  
    이런 클래스가 반복 생성된다고 생각하면 아찔하다...  
*/  
class LargeOuter {  
    private byte[] bigData = new byte[10_000_000];  // 10MB    
    class Inner {        
	    String info = "작은 정보";  
    }
}  
```  
1번의 경우는 대용량 데이터를 처리하는 때가 아니라면 만나는 건 쉽지않다. 사실 2번도 아직까지는 경험해 본적이 없다.
IDE 단에서 애초에 경고를 해주기 때문에 방어적 코딩을 하게 되니깐 정말 필요할 때가 아니라면 inner는 잘 안 쓰기 때문에 메모리 누수 문제로 낭패를 겪은 적은 없었다.
  

  
## static nested class는 주로 어떤 때에 사용하는가?  

외부 클래스와 내부 클래스가 논리적으로 연관되어 있지만, 외부 인스턴스와는 독립적인 클래스가 필요할 때 사용한다.  사실 알게 모르게 사용하고 있던 `@Builder`가 static nested class의 대표적인 좋은 예시다. 

```java  
/**  
    static nested class*/  
public class Person {  
    private String name;    
    private String age;  
    
    static public class Builder {      
		private String name;       
		private String age;  
       
        public Builder name(String name) {     
			this.name = name;          
			return this;        
		 }  
		 
        public Builder age(String age) {            
			this.age = age;            
			return this;        
        }  
        
        public Person build() {
			return new Person(this);
		}    
	}  
	
	private Person(Builder builder) { 
		this.name = builder.name;        
		this.age = builder.age;    
	}
}
  
// 사용  
Person person = new Person.Builder()  
                            .name("홍길동")  
                            .age("20")                            
                            .build();  
```  
  
  
### 근데 static이라면 클래스가 싱글턴이냐?  
static 변수 선언을 했다면 생성된 객체는 힙 메모리에 올라가고, 변수는 공유 자원에 올라가서 모든 스레드가 변수를 공유하고 접근이 가능하다.  그렇다면 클래스가 static으로 선언되어 있다면 모두가 공유할 수 있는가 싶었는데 그건 아니었다.  
사실상 이름만 static이지 변수 static선언과는 전혀 상관이 없다.  
  
``` java  
public class Outer {  
    public static class Inner {}
}  
  
// 사용  
Outer.Inner inner = new Outer.Inner(); // 이 순간에 메모리에 올라간다.  
```  
  
`inner` 변수는 static으로 선언되지 않았으므로 지역 변수이기 때문에 다른 스레드에서 공유할 수 없다.  
고로, `new Outer.Inner();` 선언 된 static 클래스에는 다른 스레드에서 접근 할 수 있는 방법이 없다.