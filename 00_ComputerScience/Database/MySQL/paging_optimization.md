# Paging Optimization

대용량 데이터에서 페이징 성능은 **OFFSET 사용 방식**에 따라 크게 달라진다.  
특히 MySQL(InnoDB)에서는 뒤 페이지로 갈수록 OFFSET 기반 조회 비용이 커지므로,  
**인덱스 활용 방식과 조회 전략을 함께 고려해야 한다.**

---

# OFFSET Pagination의 문제

일반적인 페이지네이션은 다음과 같이 구현된다.

```sql
SELECT *
FROM product
ORDER BY id LIMIT 20
OFFSET 100000;
```

이 방식은 앞의 100000건을 읽고 버린 뒤 다음 20건을 반환한다.

즉, OFFSET이 커질수록 다음 문제가 발생한다.

- 불필요한 Row Scan 증가
- 정렬 비용 증가
- 응답 시간 증가

따라서 대용량 데이터에서는 단순 OFFSET 방식이 비효율적이다.

---

## MySQL(InnoDB)와 페이징

InnoDB는 Clustered Index 기반 저장 구조를 사용한다.

- Primary Key 기준으로 데이터가 정렬되어 저장된다.
- Secondary Index는 실제 Row Data가 아니라 Primary Key 값을 참조한다.

따라서 정렬 기준과 인덱스 구조에 따라 페이징 성능 차이가 크게 난다.

## 1. Keyset Pagination

Keyset Pagination은 OFFSET 대신 마지막으로 조회한 Key 값을 기준으로 다음 페이지를 조회하는 방식이다.

예시

```sqlSELECT *
FROM product
WHERE id > 100000
ORDER BY id
LIMIT 20;
```

이 방식은 OFFSET 100000처럼 앞 데이터를 모두 읽고 버리지 않고, 마지막 Key 이후 구간만 바로 탐색할 수 있다.

### 장점

- OFFSET 스캔 제거
- 대용량 데이터에서 성능이 안정적
- 무한 스크롤, 더보기 방식에 적합

### 단점

- page=7 같은 임의 페이지 이동이 어렵다.
- 이전/다음 또는 순차 탐색에 적합하다.

## 2. Offset + Covering Index 방식

페이지 번호 기반 UI를 유지해야 하는 경우,  
단순 OFFSET 대신 인덱스만 먼저 읽고 실제 데이터는 나중에 조회하는 방식으로 최적화할 수 있다.

```sql
SELECT *
FROM academy
ORDER BY id LIMIT 10000, 10;
```

이 경우 앞의 10000개의 데이터를 모두 읽고 버리기 때문에 성능이 저하된다.  
이를 개선하는 방법 중 하나가 Covering Index + PK 조회 분리 방식.

이를

```sql
SELECT id
FROM academy
ORDER BY id LIMIT 10000, 10;
```

이렇게 먼저 PK만 조회한 후,

```sql
// 2차 쿼리
SELECT *
FROM academy
WHERE id IN (...)
```

조회된 PK로 실제 데이터를 조회하여 데이터 접근을 최소화 할 수 있다.

## No Offset Pagination

내부적으로 Keyset Pagination과 동일한 개념

```sql
SELECT *
FROM product
WHERE id > :lastId
ORDER BY id LIMIT 20;
```

보통 다음과 같은 화면에서 사용된다.

- 무한 스크롤
- 더보기 버튼
- 최신순 목록 조회