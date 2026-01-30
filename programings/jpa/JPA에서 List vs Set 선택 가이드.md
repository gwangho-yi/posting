---
created: 2025-01-30
tags:
  - jpa
  - hibernate
  - collection
---

# JPA에서 List vs Set 선택 가이드

## 핵심 요약

**기본은 Set, 순서가 정말 필요할 때만 List를 사용한다.**

JPA에서 엔티티 필드에 List vs Set을 선택하는 건 단순 컬렉션 선택이 아니라 **Hibernate의 변경 감지 메커니즘과 직결**되는 중요한 결정이다.

## List vs Set 기본 차이

### List의 특성
- **순서를 유지**한다 (`@OrderColumn` 또는 `@OrderBy`)
- **중복 엔티티를 허용**한다
- **지연 로딩 시 PersistentBag** 또는 **PersistentList** 사용
- **변경 감지가 비효율적**일 수 있다

### Set의 특성
- **순서를 보장하지 않는다**
- **중복 엔티티를 자동 제거**한다 (equals/hashCode 기반)
- **지연 로딩 시 PersistentSet** 사용
- **변경 감지가 효율적**이다

## 언제 무엇을 사용할까?

### Set 사용 (권장 - 90%)

```java
@Entity
public class ShippingOrder {
    
    // 일반적인 연관관계
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL,
               orphanRemoval = true)
    private Set<CargoItem> cargoItems = new HashSet<>();
    
    // 다대다 관계
    @ManyToMany
    @JoinTable(name = "order_ports")
    private Set<Port> visitedPorts = new HashSet<>();
    
    // 편의 메서드
    public void addCargoItem(CargoItem item) {
        cargoItems.add(item);
        item.setOrder(this);
    }
}
```

**사용 시기:**
- 일반적인 @OneToMany 관계
- @ManyToMany 관계
- 순서가 중요하지 않은 경우
- 성능이 중요한 경우

### List 사용 (특수 케이스 - 10%)

```java
@Entity
public class Container {
    
    // 적재 순서가 중요 - List + @OrderColumn
    @OneToMany(mappedBy = "container")
    @OrderColumn(name = "stack_order")
    private List<Package> packages = new ArrayList<>();
    
    // 시간순 정렬이 필요 - List + @OrderBy
    @OneToMany(mappedBy = "container")
    @OrderBy("timestamp ASC")
    private List<StatusHistory> history = new ArrayList<>();
}
```

**사용 시기:**
- 순서가 비즈니스 로직상 중요한 경우
- 시간순 정렬이 필요한 경우

## Hibernate의 내부 처리 차이

### 1. 내부 컬렉션 래퍼

Hibernate는 선언한 컬렉션을 자신의 래퍼로 감싼다:

```java
// Set → PersistentSet으로 래핑
private Set<OrderItem> items = new HashSet<>();

// List (@OrderColumn 없음) → PersistentBag으로 래핑
private List<OrderItem> items = new ArrayList<>();

// List (@OrderColumn 있음) → PersistentList로 래핑
@OrderColumn(name = "position")
private List<OrderItem> items = new ArrayList<>();
```

### 2. 변경 감지(Dirty Checking) 차이

#### Set - 효율적 

```java
@Entity
public class Order {
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL,
               orphanRemoval = true)
    private Set<OrderItem> items = new HashSet<>();
}

// 아이템 하나 제거
Order order = entityManager.find(Order.class, 1L);
OrderItem toRemove = order.getItems().iterator().next();
order.getItems().remove(toRemove);

entityManager.flush();
```

**실행 SQL:**
```sql
-- 딱 1개만 삭제!
DELETE FROM order_item WHERE id = ?
```

**이유:**
- PersistentSet은 **스냅샷**을 유지
- 현재 Set과 스냅샷을 **equals/hashCode로 비교**
- 정확히 무엇이 제거/추가되었는지 파악 가능

#### List (@OrderColumn 없음) - 비효율적

```java
@Entity
public class Order {
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL,
               orphanRemoval = true)
    private List<OrderItem> items = new ArrayList<>();
}

// 같은 시나리오
Order order = entityManager.find(Order.class, 1L);
order.getItems().remove(0);

entityManager.flush();
```

**실행 SQL:**
```sql
-- 전부 삭제하고 남은 것들 다시 INSERT!
DELETE FROM order_item WHERE order_id = ?
INSERT INTO order_item VALUES (?, ?, ?)
INSERT INTO order_item VALUES (?, ?, ?)
```

#### List (@OrderColumn 있음) - 순서 재정렬 필요

```java
@Entity
public class Order {
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL,
               orphanRemoval = true)
    @OrderColumn(name = "item_order")
    private List<OrderItem> items = new ArrayList<>();
}

// 중간에 삽입
Order order = entityManager.find(Order.class, 1L);  // [A, B, C]
OrderItem newItem = new OrderItem("D");
order.getItems().add(1, newItem);  // [A, D, B, C]

entityManager.flush();
```

**실행 SQL:**
```sql
-- 순서 재정렬을 위한 추가 UPDATE
INSERT INTO order_item (order_id, item_order, ...) VALUES (1, 1, ...)
UPDATE order_item SET item_order = 2 WHERE id = ?  -- B 순서 변경
UPDATE order_item SET item_order = 3 WHERE id = ?  -- C 순서 변경
```

### 3. List가 전부 삭제 후 재생성하는 이유

```java
// 초기 상태
Order order = entityManager.find(Order.class, 1L);
// items = [사과(id=10), 바나나(id=11), 사과(id=12)]  // 중복!

// 하나 제거
order.getItems().remove(0);  // 첫 번째 "사과" 제거

// flush() 시점
entityManager.flush();

// Hibernate의 고민:
// 스냅샷(DB): [사과(10), 바나나(11), 사과(12)]
// 현재(메모리): [바나나(11), 사과(12)]
//
// "어떤 사과가 삭제되었지?"
// → 사과(10)? 사과(12)? 구분 불가!
```

**문제점:**
1. **순서 정렬 키가 없음** - DB에 position 컬럼 없음
2. **중복 저장 가능** - 같은 객체가 여러 개
3. **매칭 불가능** - equals/hashCode 없으면 객체 동일성 판단 불가
4. **안전한 선택** - 전부 삭제 후 재생성

**Hibernate의 내부 처리:**
```java
// Hibernate 내부 로직 (단순화)
public void updateBag(PersistentBag bag) {
    List snapshot = bag.getStoredSnapshot();
    List current = bag.getCurrentElements();
    
    // 누가 삭제되었는지 확신할 수 없다!
    // 안전하게 전부 삭제 후 현재 상태를 다시 INSERT
    deleteAll(snapshot);
    insertAll(current);
}
```

### 4. Set은 왜 정확히 알 수 있나?

```java
@Entity
public class OrderItem {
    @Id
    @GeneratedValue
    private Long id;
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof OrderItem)) return false;
        OrderItem that = (OrderItem) o;
        return Objects.equals(id, that.id);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(id);
    }
}
```

**equals/hashCode 덕분에:**
- 스냅샷과 현재 상태를 **정확히 비교** 가능
- "사과(id=10)이 삭제되었다"를 확신할 수 있음
- 필요한 SQL만 정확히 실행

## 성능 비교

### 시나리오: 10개 아이템 중 1개 제거

| 타입 | SQL 개수 | 비고 |
|------|----------|------|
| **Set** | 1개 | `DELETE` 1번 |
| **List (@OrderColumn 없음)** | 11개 | `DELETE` 1번 + `INSERT` 10번 |
| **List (@OrderColumn 있음)** | 10개 | `DELETE` 1번 + `UPDATE` 9번 (순서 재정렬) |

## equals/hashCode의 중요성

Set을 사용할 때는 **반드시 올바른 equals/hashCode 구현**이 필요하다:

```java
@Entity
public class OrderItem {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String itemCode;
    
    // 비즈니스 키 사용 권장
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof OrderItem)) return false;
        OrderItem that = (OrderItem) o;
        return Objects.equals(itemCode, that.itemCode);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(itemCode);
    }
}
```

### ⚠️ ID 기반 equals/hashCode의 위험

```java
// ❌ ID 기반은 위험
@Override
public boolean equals(Object o) {
    return Objects.equals(id, ((OrderItem) o).id);
}

// 문제 상황:
OrderItem item = new OrderItem();  // id = null
Set<OrderItem> items = new HashSet<>();
items.add(item);  // hashCode 계산

entityManager.persist(item);  // id = 1 할당됨

// hashCode가 변경되어 Set에서 찾을 수 없게 됨!
items.contains(item);  // false 반환 가능
```

## 실전 선택 기준

| 상황 | 선택 | 이유 |
|------|------|------|
| 일반적인 @OneToMany | **Set** | 순서 불필요, 중복 방지, 성능 좋음 |
| @ManyToMany | **Set** | 관계의 특성상 중복 의미 없음 |
| 순서가 중요한 데이터 | **List + @OrderColumn** | 순서 정보 저장 필요 |
| 정렬만 필요 | **List + @OrderBy** | 조회 시에만 정렬 |
| 성능이 중요 | **Set** | 변경 감지 효율적 |

## 정리

### Hibernate 처리 방식 비교

| 측면 | Set | List (no @OrderColumn) | List (@OrderColumn) |
|------|-----|------------------------|---------------------|
| 래퍼 타입 | PersistentSet | PersistentBag | PersistentList |
| 변경 감지 | ✅ 효율적 | ❌ 비효율적 | ⚠️ 중간 |
| 중복 처리 | 자동 제거 | 허용 | 허용 |
| SQL 개수 | 최소 | 최다 | 중간 |
| 순서 보장 | ❌ | ❌ | ✅ |
| 성능 | ✅ 최고 | ❌ 최악 | ⚠️ 중간 |

### 핵심 원칙

1. **기본은 Set** - 특별한 이유 없으면 Set 사용
2. **순서가 정말 필요할 때만 List** - 비즈니스 로직상 순서가 의미 있을 때
3. **equals/hashCode 필수** - Set 사용 시 반드시 구현
4. **비즈니스 키 사용** - ID보다는 불변 비즈니스 키로 동일성 판단

## 실무 예시

```java
// 물류 시스템 예시
@Entity
public class ShippingOrder {
    
    // ✅ Set: 주문 아이템 (순서 불필요)
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL,
               orphanRemoval = true)
    private Set<OrderItem> items = new HashSet<>();
    
    // ✅ Set: 방문 항구 (중복 불필요)
    @ManyToMany
    private Set<Port> visitedPorts = new HashSet<>();
    
    // ✅ List + @OrderColumn: 경유지 순서 (순서 중요)
    @OneToMany(mappedBy = "order")
    @OrderColumn(name = "route_order")
    private List<RouteStop> route = new ArrayList<>();
    
    // ✅ List + @OrderBy: 상태 이력 (시간순 정렬)
    @OneToMany(mappedBy = "order")
    @OrderBy("createdAt DESC")
    private List<StatusHistory> statusHistory = new ArrayList<>();
}
```

## 참고 자료

- Hibernate 공식 문서: Collections Mapping
- JPA 2.x 스펙: Collection-valued Association
