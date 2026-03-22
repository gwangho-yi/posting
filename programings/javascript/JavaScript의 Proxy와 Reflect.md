
자바도 존재하고 자바스크립트에도 존재하는 Proxy는 언어 동작 레벨에서 자바와는 어떤 차이로 동작하는지 궁금해서 알아봤다. 그리고 한 가지 특이한 점을 발견했기도 했고, 자바 진영에서는 흔히 스프링 AOP나 Jpa를 배울 때 Proxy를 접했는데 최근에 프론트를 자주 만지는 상황이라 이번 기회에 JS도 한 번 파고 들어서 깊이를 더해보고자 한다.


[MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Proxy)에서는 Proxy에 대해 다음과 같이 설명한다.

>한 객체에 대한 기본 작업을 가로채고 재정의하는 프록시를 만들 수 있습니다

그리고 Proxy라는 단어의 사전적 의미는 다음과 같다. 
**대리인** , **대리**

즉, *특정 기본 작업을 가로채고 재정의 하는 대리자* 라고 해석할 수 있다.

그렇다면 이 대리자를 어떤 때에 사용할 수 있을까?
백엔드에서는 흔히 로그를 구현할 때 사용한다. 특정 메서드가 호출되었을 때, 실행된 내용을 로깅할 때 사용한다.
프론트에서는 상태 변화를 감지할 때 사용한다. 데이터 변경을 가로채서 화면을 갱신하는데 사용한다.

즉, 두 상황 모두 Proxy는 기본 기능에 더해 **부가적인 역할**을 수행하는 것이다.
기본적인 기능도 아예 다른 기능으로 작동하게 하는 것도 할 수 있지만, 주로 Proxy를 사용한다고 하면 대체로 원래동작은 유지하면서 부가 동작을 끼워넣는 패턴을 사용한다.


우선 원본 기능을 만든 후에 Proxy를 만들어보자. 타켓으로 사용할 기본 객체다.
```javascript
const target = { 
	name: "환경설정", 
	age: 20, 
}; 

console.log(target.name); // 환경설정
console.log(target.age); // 20
```



그렇다면 Proxy를 한 번 만들어보자. Proxy는 두 가지 매개변수를 사용하여 생성한다.
- `target`: 프록시 할 원본 객체
- `handler`: 가로채는 작업과 가로채는 작업을 재정의하는 방법을 정의하는 객체, 트랩이라고도 불린다.

트랩은 간단하게 getter와 setter가 실행될 때, 로그가 작동 되도록 설정했다.
```javascript
const target = { name: '환경설정', age: 20 }; 

const handler = { 
	get(target, prop, receiver) { // 읽기
	 console.log(`${prop} 프로퍼티 읽기`); 
	 return target[prop];
}, 

	set(target, prop, value, receiver) { // 쓰기
	 console.log(`${prop} = ${value} 쓰기`); 
	 target[prop] = value; 
	 return true; // 성공 여부 반환 
	} 
}; 
const proxy = new Proxy(target, handler); 

proxy.name; // "name 프로퍼티 읽기" → '환경설정' 
proxy.age = 30; // "age = 30 쓰기"
```


getter 와 setter 실행 시, 트랩에 심어 두었던 로그 함수가 작동된다. getter, setter 말고도 객체를 다루는 다른 함수들도 가로채서 설정 가능하다.


중요한건 이 지점부터다. 이 글을 쓰는 이유는 사실 `Reflect` 때문이다. Mdn에 Proxy에 관한 문서를 읽어내려가다 보면 아래 문장을 발견할 수 있다.
>[`Reflect`](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Reflect) 클래스의 도움으로 일부 접근자에게 원래 동작을 제공하고 다른 접근자를 재정의할 수 있습니다.
이게 무슨 말일까?

우선 연달아 나오는 `Reflect`의 쓰임새부터 살펴보자면,
```javascript
const target = {
  message1: "hello",
  message2: "everyone",
};

const handler = {
  get(target, prop, receiver) {
    if (prop === "message2") {
      return "world";
    }
    return Reflect.get(...arguments);
  },
};

const proxy = new Proxy(target, handler);

console.log(proxy.message1); // hello
console.log(proxy.message2); // world
```

첫 번째 예제에서는 getter에서  `target[prop]`를 반환했으나, 이번 예제에서는`Reflect.get(...arguments)`를 반환하고 있다. 하지만 결과는 동일하다. 결과는 동일한데 이건 왜 사용하는걸까?
Reflect의 정의는 다음과 같이 정리되어 있다.

> **`Reflect`** 는 중간에서 가로챌 수 있는 JavaScript 작업에 대한 메서드를 제공하는 내장 객체입니다. 메서드의 종류는 [프록시 처리기](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Proxy/Proxy)와 동일합니다

즉, **Proxy에 대한 메서드를 제공하는 내장 객체**라는 뜻이다.

내용을 정리 했음에도 Proxy만큼 와닿지는 않는다. 애초에 `Reflect`는 왜 필요한걸까? 이 의문을 해결하기 위해서는 `Reciever`전파에 대해 알고 있어야한다.

