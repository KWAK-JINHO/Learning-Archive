# JPA Performance Optimization

JPA는 생산성을 높이는 ORM 프레임워크이지만,  
대용량 데이터 조회나 벌크 연산에서는 성능 이슈가 발생할 수 있다.

따라서 JPA 환경에서는 **DB 튜닝뿐 아니라 ORM 레벨 최적화도 함께 고려해야 한다.**

---

## DTO 조회 전략

엔티티를 그대로 조회하면 다음과 같은 성능 문제가 발생할 수 있다.

### 1. Hibernate 캐시 오버헤드

Hibernate는 다음과 같은 캐시 구조를 가진다.

- 1차 캐시 (Persistence Context)
- 2차 캐시

엔티티 조회 시 캐시 관리와 일관성 유지 비용이 발생할 수 있다.

### 2. 불필요한 컬럼 조회

엔티티를 조회하면 **모든 필드가 조회된다.**
예를 들어 `SELECT *` 와 같은 경우. 하지만 실제로는 일부 컬럼만 필요한 경우가 많다.

이런 경우

- 데이터 전송량 증가
- 메모리 사용량 증가

문제가 발생할 수 있다.

### 3. N+1 문제

연관 엔티티를 조회할 때 다음과 같은 문제가 발생할 수 있다.

`게시글 10개 조회 → 작성자 10번 조회`

특히 다음 상황에서 자주 발생한다.

- Lazy Loading
- OneToMany / ManyToOne 관계 조회

### 4. Distinct 사용 시 비효율

엔티티 조회에서 DISTINCT를 사용하면 엔티티의 모든 컬럼이 DISTINCT 대상이 된다.  
이 경우

- 임시 테이블 생성
- 정렬 비용 증가

등의 성능 문제가 발생할 수 있다.

---

## DTO 직접 조회

대용량 조회나 성능이 중요한 경우 DTO Projection 방식이 유리할 수 있다.

예시

```sql
SELECT new ProductDto(p.id, p.name)
FROM Product p
```

### 장점

- 필요한 컬럼만 조회
- 메모리 사용량 감소
- 네트워크 비용 감소

---

## 상황별 전략

### 엔티티 조회가 적합한 경우

- 실시간 데이터 수정
- 변경 감지(Dirty Checking) 필요
- 트랜잭션 내부 데이터 관리 필요

### DTO 조회가 적합한 경우

- 조회 전용 API
- 대용량 데이터 조회
- 읽기 중심 서비스

---

## ORM 환경 최적화 전략

### 1. 조회 컬럼 최소화

필요한 컬럼만 조회하여 데이터 전송량을 줄인다.

예시
`select product.id, product.name`

### 2. OneToOne 관계 주의

OneToOne 관계는 Lazy 로딩이 제대로 동작하지 않는 경우가 있다.

이 경우 join fetch 사용, 별도 조회 전략을 고려해야 한다.

### 3. 연관 관계 최소 조회

연관 엔티티 전체를 조회하기보다 ID만 조회하는 방식이 효율적일 수 있다.

예시
`select order.user.id`

---

## Bulk Update 전략

JPA는 기본적으로 Dirty Checking 방식으로 업데이트를 수행한다.

즉,

1. 엔티티 조회
2. 변경 감지
3. UPDATE 수행 이 방식은 편리하지만 대량 데이터 처리에서는 비효율적일 수 있다.

### Dirty Checking 방식

#### 장점

- 트랜잭션 관리 편리
- 변경 감지 자동 처리

#### 단점

- 많은 엔티티 조회 필요
- 메모리 사용량 증가

### Bulk Update

대량 업데이트는 다음 방식이 더 효율적일 수 있다.  
`queryFactory.update(entity)` 또는 `JPQL update`

#### 주의점

- 영속성 컨텍스트와 동기화되지 않는다.
- 캐시 정리가 필요할 수 있다.

---

## Bulk Insert

JPA는 Bulk Insert에 최적화되어 있지 않다.

이유

- ORM 특성상 개별 insert 수행
- JDBC Batch와 비교하면 성능이 떨어질 수 있음

### JDBC Batch

JDBC에서는 다음 옵션을 통해 Batch Insert 성능을 개선할 수 있다.

`rewriteBatchedStatements=true`

#### 장점

- Insert SQL 병합
- 네트워크 비용 감소

### Querydsl Bulk Insert

Querydsl-SQL을 사용하면 Native SQL 기반 Bulk Insert를 수행할 수 있다.

하지만 다음과 같은 단점이 있다.

- 설정 복잡
- 스키마 기반 코드 생성 필요