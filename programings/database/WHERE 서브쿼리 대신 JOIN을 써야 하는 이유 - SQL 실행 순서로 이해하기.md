생각보다 많은 사람들이 DB의 `JOIN`과 `WHERE` 순서를 헷갈려한다. 특히 `WHERE`에 서브쿼리를 넣어서 데이터를 조회하는 경우가 많은데, 이게 왜 문제가 되는 걸까??

## SQL은 작성 순서대로 실행되지 않는다

SQL을 처음 배울 때 보통 `SELECT`부터 작성한다. 그런데 실제 DB 엔진이 쿼리를 처리하는 순서는 작성 순서와 완전히 다르다.

```
작성 순서:  SELECT → FROM → JOIN → WHERE → GROUP BY → HAVING → ORDER BY
실행 순서:  FROM → JOIN → WHERE → GROUP BY → HAVING → SELECT → ORDER BY
```

핵심은 `FROM`과 `JOIN`이 **가장 먼저** 실행되어 작업할 데이터셋을 만들고, `WHERE`는 그 **이후에** 필터링한다는 것이다.

이걸 이해하면 왜 `WHERE`에 서브쿼리를 넣으면 안 되는지 자연스럽게 알 수 있다.

## WHERE 서브쿼리의 문제

예를 들어, 주문 테이블에서 VIP 고객의 주문만 조회한다고 해보자.

### WHERE 서브쿼리 방식

```sql
SELECT o.order_id, o.customer_id, o.amount
FROM orders o
WHERE o.customer_id IN (
    SELECT c.customer_id
    FROM customers c
    WHERE c.grade = 'VIP'
);
```

실행 흐름을 따라가보면 이렇다.

1. `FROM orders` → orders 테이블 전체를 읽는다
2. `WHERE` 평가 시점에서 **각 행마다** 서브쿼리가 평가될 수 있다
3. 서브쿼리는 매번 customers 테이블을 탐색한다

문제가 보이는가? `WHERE`는 실행 순서상 **행 단위 필터링** 단계다. 여기에 서브쿼리를 넣으면 "이미 만들어진 데이터셋에서 하나씩 조건을 검사"하는 구조가 된다. 옵티마이저가 알아서 최적화해주는 경우도 있지만, 본질적으로 비효율적인 구조다.

특히 상관 서브쿼리 Correlated Subquery가 되면 상황이 더 나빠진다.

```sql
SELECT o.order_id, o.customer_id, o.amount
FROM orders o
WHERE EXISTS (
    SELECT 1
    FROM customers c
    WHERE c.customer_id = o.customer_id  -- 외부 쿼리 참조
    AND c.grade = 'VIP'
);
```

외부 쿼리의 `o.customer_id`를 참조하기 때문에 orders의 **모든 행마다** 내부 쿼리가 반복 실행된다. orders가 100만 건이면 내부 쿼리도 100만 번 실행될 수 있다는 뜻이다.

### JOIN 방식

```sql
SELECT o.order_id, o.customer_id, o.amount
FROM orders o
INNER JOIN customers c ON o.customer_id = c.customer_id
WHERE c.grade = 'VIP';
```

같은 결과인데 실행 흐름이 근본적으로 다르다.

1. `FROM orders` → orders 테이블 읽음
2. `JOIN customers` → 실행 순서상 WHERE보다 **먼저** 두 테이블을 결합
3. `WHERE c.grade = 'VIP'` → 이미 결합된 결과셋에서 단순 필터링

`JOIN`은 DB 엔진이 **Hash Join**, **Merge Join**, **Nested Loop** 같은 최적화된 알고리즘 중에서 상황에 맞는 걸 골라서 실행한다. 인덱스도 효과적으로 활용할 수 있다.

반면 `WHERE` 서브쿼리는 옵티마이저가 내부적으로 `JOIN`으로 변환(query rewriting)하지 못하면 그대로 비효율적으로 실행된다.

## 왜 이런 차이가 생기는 걸까??

결국 SQL 실행 순서 때문이다.

- `JOIN`은 **데이터셋을 구성하는 단계**에서 동작한다. 두 테이블의 관계를 먼저 파악하고, 최적의 결합 전략을 세울 수 있다.
- `WHERE`는 **이미 만들어진 데이터셋을 걸러내는 단계**에서 동작한다. 여기에 서브쿼리를 넣으면 필터링할 때마다 별도 조회가 발생하는 구조가 된다.

비유하자면, `JOIN`은 두 팀을 **한 방에 모아놓고** 짝을 맞추는 거고, `WHERE` 서브쿼리는 한 명씩 불러서 **옆방에 가서 짝이 있는지 확인하고 오라**고 하는 거다.

## 정리

| 구분 | WHERE 서브쿼리 | JOIN |
|------|---------------|------|
| 실행 시점 | 필터링 단계 (행 단위) | 데이터셋 구성 단계 |
| 최적화 | 옵티마이저 의존 (변환 실패 가능) | DB 엔진이 직접 최적 알고리즘 선택 |
| 인덱스 활용 | 제한적 | 효과적 |
| 상관 서브쿼리 시 | 외부 행마다 반복 실행 | 해당 없음 |
| 가독성 | 중첩이 깊어지면 읽기 어려움 | 테이블 관계가 명시적 |

데이터를 합치는 일은 합치는 단계(`JOIN`)에서, 거르는 일은 거르는 단계(`WHERE`)에서 하는 것이 SQL 실행 순서에 맞는 자연스러운 설계다.

물론 `EXISTS`를 사용한 서브쿼리가 대량 테이블에서 존재 여부만 확인할 때 오히려 유리한 케이스도 있긴 하다. 하지만 일반적인 데이터 조회에서는 `JOIN`이 거의 항상 더 나은 선택이다.
