테이블 안에서 kg을 입력하면 pund 값을 자동으로 계산해주는 기능을 만들던 중에 의문이 들어서 잠깐 정리를 한다. 당연하다는 듯이 `watch`를 썼는데, 돌아보니 이건 Vue의 설계 의도와 맞지 않는 코드였다.

## 무엇이 문제일까?

내가 짠 코드는 이런 구조였다.

```js
const kg = ref(0)
const bl = ref(0)

watch(kg, (newVal) => {
  bl.value = newVal * 2.20462
})
```

동작은 한다. 근데 뭔가 찜찜했다. `watch`가 맞는 선택인지 확신이 없었다.

## computed vs watch, 진짜 차이

`computed`와 `watch`를 구분하는 기준은 생각보다 단순하다.

| | computed | watch |
|---|---|---|
| 목적 | 값 계산 및 반환 | 사이드 이펙트 실행 |
| 반환값 | 있음 | 없음 |
| 캐싱 | O | X |
| 비동기 | 불가 | 가능 |
| 이전 값 접근 | 불가 | 가능 (oldVal) |

핵심은 이 질문 하나로 정리된다.

> "템플릿에서 사용할 값을 계산하는가?" → `computed`
> "데이터 변화에 반응해서 무언가를 실행하는가?" → `watch`

kg → bl 변환은 단순한 동기 파생값 계산이다. 비동기도 없고, 외부 API 호출도 없다. 그러니 `computed`가 맞다.

```js
const kg = ref(0)
const bl = computed(() => kg.value * 2.20462)
```

훨씬 깔끔하고, `bl`이 별도의 `ref`로 떠돌지 않는다.

## computed의 캐싱은 어떻게 동작할까?

`computed`의 캐싱을 처음 들으면 막연하게 느껴진다. "이전 값과 비교해서 같으면 캐시를 쓴다"고 생각하기 쉬운데, 실제로는 다르다.

Vue 내부적으로는 `dirty` 플래그로 관리한다.

```
의존 ref가 set됨 → dirty = true → 다음 접근 시 재계산
의존 ref가 그대로 → dirty = false → 캐시된 값 반환
```

**값을 비교하는 게 아니다.** 의존하고 있는 `ref`가 `.value = 새값`으로 set된 순간 `dirty = true`가 된다.

```js
const kg = ref(10)
const bl = computed(() => kg.value * 2.20462)

console.log(bl.value) // 계산 실행 → 22.0462
console.log(bl.value) // 캐시 반환 → 22.0462 (재계산 없음)

kg.value = 10        // 같은 값이어도 set 자체가 발생하진 않음
console.log(bl.value) // 캐시 반환

kg.value = 20        // set 발생 → dirty = true
console.log(bl.value) // 재계산 → 44.0924
```

그래서 `computed`를 템플릿에서 여러 번 참조해도 실제 계산은 한 번만 일어난다.

## 그럼 캐싱 판단은 어느 시점에 일어나나?

input에 값을 타이핑하다 보면 궁금증이 생긴다. "포커스 아웃될 때 계산되는 건가?"

아니다. `v-model`의 기본 동작을 보면 이해가 된다.

```html
<!-- 이 코드는 -->
<input v-model="kg" />

<!-- 사실상 이것과 같다 -->
<input :value="kg" @input="kg = $event.target.value" />
```

`@input` 이벤트는 타이핑할 때마다 발생한다. 즉 키를 누를 때마다 `kg.value`가 set되고, `dirty = true`가 된다. 포커스 아웃을 기다리지 않는다.

만약 포커스 아웃 시점에만 반응하게 하고 싶다면 `@change`를 쓰면 된다.

```html
<input :value="kg" @change="kg = $event.target.value" />
```

`@change`는 포커스 아웃이나 Enter 시점에만 발생하기 때문에 그때만 ref가 업데이트된다.

## 정리

| 상황 | 선택 |
|---|---|
| 단순 동기 파생값 계산 | `computed` |
| 비동기 API 호출 후 다른 ref 갱신 | `watch` |
| 이전 값과 새 값을 비교해야 할 때 | `watch` |
| 템플릿에서 바로 사용할 값 | `computed` |

`watch` 안에서 다른 `ref`를 재계산하는 패턴 자체가 나쁜 건 아니다. 다만 그 재계산이 단순한 동기 파생값이라면 `computed`가 더 적합하다. 나처럼 kg → bl 같은 단순 변환에 `watch`를 쓰고 있었다면, `computed`로 교체하는 게 Vue의 설계 의도에 맞다.
