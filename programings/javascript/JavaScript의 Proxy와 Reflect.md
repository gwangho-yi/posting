
자바도 존재하고 자바스크립트에도 존재하는 Proxy는 언어 동작 레벨에서 자바와는 어떤 차이로 동작하는지 궁금해서 알아봤다. 그리고 한 가지 특이한 점을 발견했기도 했고, 자바 진영에서는 흔히 스프링 AOP나 Jpa를 배울 때 Proxy를 접했는데 최근에 프론트를 자주 만지는 상황이라 이번 기회에 JS도 한 번 파고 들어서 깊이를 더해보고자 한다.

## Proxy란?

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

## Proxy 만들어보기

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

## Reflect의 등장

중요한건 이 지점부터다. 이 글을 쓰는 이유는 사실 `Reflect` 때문이다. Mdn에 Proxy에 관한 문서를 읽어내려가다 보면 아래 문장을 발견할 수 있다.

>[`Reflect`](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Reflect) 클래스의 도움으로 일부 접근자에게 원래 동작을 제공하고 다른 접근자를 재정의할 수 있습니다.

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

"원래 동작을 제공한다"는 것은 **Proxy가 가로채지 않았을 때와 똑같이 동작하게 한다**는 뜻이다. `get` 트랩을 정의한 순간 모든 프로퍼티 접근이 이 함수를 통과하는데, `message1`처럼 커스텀 동작이 필요 없는 경우에 `Reflect.get(...arguments)`가 "프록시가 없었던 것처럼 원본 객체에서 값을 꺼내라"는 역할을 한다.

즉 MDN의 그 문장을 풀어쓰면, **message2는 커스텀 동작으로 재정의하고, 나머지(message1 등)는 Reflect를 통해 프록시가 없을 때와 동일하게 동작시킬 수 있다**는 뜻이다.

Reflect의 정의는 다음과 같이 정리되어 있다.

> **`Reflect`** 는 중간에서 가로챌 수 있는 JavaScript 작업에 대한 메서드를 제공하는 내장 객체입니다. 메서드의 종류는 [프록시 처리기](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Proxy/Proxy)와 동일합니다

즉, **Proxy에 대한 메서드를 제공하는 내장 객체**라는 뜻이다.

내용을 정리 했음에도 Proxy만큼 와닿지는 않는다. 애초에 `Reflect`는 왜 필요한걸까? 이 의문을 해결하기 위해서는 `receiver` 전파에 대해 알고 있어야한다.

## receiver 전파

먼저 `receiver` 란, **프로퍼티 접근을 실제로 시작한 객체**이다.
handler 메서드를 분해하면서 새로운 예제를 보자.

```javascript
const parent = {  
    _name: '부모',  
    get name() {  // 원본 getter
        return this._name; 
    }  
};  

const handler = { 
	get(
		target,   // 프록시가 감싸고 있는 원본 객체 
		prop,     // 접근하려는 프로퍼티 이름
		receiver  // .prop을 실제로 호출한 객체 
	) {
		return target[prop];
	} 
}
  
const proxy = new Proxy(parent, handler);

const child = Object.create(proxy); // child → proxy → parent

child._name = '자식';

console.log(child.name); // '부모'
```

기존 예제와는 별로 다를게 없지만 상속 관계에서 `target[prop]`의 호출에 유의해서 봐야한다. log를 호출했을 때 원래 의도는 **자식**이 출력되길 의도한 코드이지만 **부모**가 출력된다. 이건 prototype의 호출 순서 때문에 발생하는 문제다. 
1. `child.name` 호출 -> `child`에는 `name` 메서드 없음 
2. `proxy`의 `get` 호출 -> `receiver` : `child`, `target`: `parent`
3. `target[prop]` -> `parent['name']` -> getter 실행
4. getter 안의 `this`가 **parent**가 됨 (parent에서 직접 꺼냈으니까)
5. `this._name` -> `parent._name` -> 부모

즉, *child가 호출했다는 정보(receiver)는 완전히 무시된다.*

`target[prop]`이라고 쓰면 "parent에서 직접 프로퍼티를 꺼내"라는 뜻이 된다. getter 입장에서 `this`는 당연히 `parent`가 되는 것이다.

## Reflect.get이 해결하는 것

```javascript
const proxy = new Proxy(parent, handler);

const handler = { 
	get(target, prop, receiver) {
		return Reflect.get(target, prop, receiver); // receiver를 전달
	} 
}

const child = Object.create(proxy);
child._name = '자식';

console.log(child.name); // '자식'
```

`Reflect.get(target, prop, receiver)`는 "target에서 prop을 찾되, getter가 있으면 **this를 receiver로 설정해라**"라는 뜻이다. 동일한 순서를 따라가면 이렇게 된다.

1. `child.name` 호출 -> proxy의 `get` 트랩 발동 -> `receiver`는 `child`
2. `Reflect.get(parent, 'name', child)` 실행
3. parent에서 name getter를 찾되, **this를 child로 설정**해서 실행
4. `this._name` -> `child._name` -> 자식

`Reflect.get`의 세 번째 인자가 "이 getter를 실행할 때 this를 이 객체로 해라"라고 지정해주는 것이다. `target[prop]`에는 이 정보를 넘길 방법이 없다.

receiver 전파란, **"원래 호출자가 누구인지"라는 정보를 getter까지 전달하는 것**을 뜻한다. `target[prop]`은 이 정보를 버리고, `Reflect.get(target, prop, receiver)`은 이 정보를 끝까지 전달한다.

## Reflect의 존재 이유 정리

Reflect는 **Proxy 트랩에서 기본 동작을 안전하게 위임(forwarding)하기 위해** 만들어졌다. MDN에서도 Reflect의 주된 사용 목적을 이렇게 명시하고 있다.

그리고 Proxy 트랩 13개와 Reflect 메서드 13개가 이름, 인자까지 정확히 1:1로 대응한다. 트랩 안에서 "원래 동작으로 위임"할 때 고민 없이 `Reflect.같은이름(...arguments)`을 쓰면 된다.

이 통일된 인터페이스가 receiver를 인자로 받을 수 있는 구조이기 때문에 안전한 전파가 가능한 것이다. 정리하면 Reflect는 다음 두 가지를 달성한다.

1. Proxy 트랩과 1:1로 대응하는 **통일된 인터페이스 제공**
2. receiver 전파를 통한 **안전한 기본 동작 위임**

이 둘은 분리된 게 아니라 연결되어 있다. 1번이 2번을 가능하게 하는 관계다.

## 프론트에서 Proxy는 어디에 쓰일까?

백엔드에서는 AOP, 트랜잭션, 지연 로딩 등에서 Proxy를 접하게 되는데, 프론트에서는 주로 다음과 같은 곳에서 사용된다.

**반응형 상태 관리** - 가장 대표적인 사용처다. Vue 3의 `reactive()`가 정확히 이 패턴이다. 객체의 `set` 트랩으로 변경을 감지하고 UI를 다시 그린다. React 생태계에서는 Valtio가 Proxy를 전면에 내세운 상태 관리 라이브러리다.

**불변성 보장** - Immer가 `produce` 안에서 변경을 가로채서 원본은 건드리지 않고 새 객체를 만들어낸다. Redux Toolkit이 내부적으로 Immer를 쓰기 때문에, RTK를 쓰고 있다면 간접적으로 Proxy를 사용하고 있는 셈이다.

**개발 도구 및 디버깅** - React가 개발 모드에서 props 변경을 감지해 경고를 띄우거나, Vue devtools가 상태 변화를 추적하는 것도 Proxy 기반이다.

결국 백엔드의 Proxy가 **"요청 흐름에 끼어들기"**가 핵심이라면, 프론트엔드의 Proxy는 **"데이터 변화에 끼어들기"**가 핵심이다. 방향이 다를 뿐 "원본을 건드리지 않고 대리인이 가로채서 부가 동작을 수행한다"는 본질은 같다.

## 정리

| 구분                              | 내용                                        |
| ------------------------------- | ----------------------------------------- |
| Proxy란                          | 객체의 기본 동작을 가로채는 대리자                       |
| 주된 용도                           | 원래 동작은 유지하면서 부가 동작을 끼워넣는 것                |
| Reflect의 존재 이유                  | Proxy 트랩에서 기본 동작을 안전하게 위임하기 위해            |
| receiver 전파                     | 프로퍼티 접근을 시작한 원래 호출자 정보를 getter까지 전달하는 것   |
| `target[prop]` vs `Reflect.get` | 전자는 receiver를 유실, 후자는 안전하게 전달             |
| 프론트에서의 활용                       | 반응형 상태 관리(Vue, Valtio), 불변성(Immer), 개발 도구 |
