# 접근 경로

Access Path는 DB가 테이블에서 데이터를 읽는 방법을 의미.

같은 SQL이라도 데이터를 찾는 방법은 여러 가지가 존재하며,  
옵티마이저(Optimizer) 는 여러 접근 방식 중 가장 비용이 낮은 방법을 선택한다.

## Table Access

### Table Full Scan

Block1 → Block2 → Block3 → Block4 순으로, 테이블의 모든 블록을 처음부터 끝까지 읽는 방식

#### 특징

- Sequential I/O 발생
- 인덱스가 없어도 동작
- 대량 데이터 조회에 유리

예시

`SELECT * FROM user;`

---

## Index Access

### Index Unique Scan

Primary Key 조회 시 사용되는 방식 정확히 한 건을 조회하는 경우로 가장 빠른 접근 방식이다.

예시

```sql
SELECT *
FROM user
WHERE id = 1;
```

Index 탐색 -> RowID 획득 -> Table Access 순으로 동작.

---

## Index Range Scan

특정 범위 조건을 조회할 때 사용

예

```sql
SELECT *
FROM user
WHERE age BETWEEN 20 AND 30;
```

### 특징

- Index를 통해 범위를 탐색
- Random I/O 발생

Index 탐색 -> RowID 획득 -> Table Access 순으로 동작

---

## Index Full Scan

인덱스를 처음부터 끝까지 읽는 방식으로 , 주로 정렬을 피하기 위해 사용.

예시

```sql
SELECT *
FROM user
ORDER BY id;
```

### 특징

- Index 전체를 순차적으로 읽음
- Sort 연산을 줄일 수 있음
- Index Skip Scan
- 복합 인덱스에서 선행 컬럼 없이 검색할 때 사용

예시

복합 인덱스  
`INDEX (gender, age)`

SQL문

```sql
SELECT *
FROM user
WHERE age = 25;
```

옵티마이저가 gender 값을 건너뛰며 탐색한다.

---

같은 SQL이라도 Access Path에 따라 성능이 크게 달라질 수 있다.