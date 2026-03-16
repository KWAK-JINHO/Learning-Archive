# SQL 처리 과정과 옵티마이저

## SQL 파싱

SQL Parser는 SQL 문을 분석하고 문법 및 의미 오류를 검사한다.

### SQL 최적화

1. 파싱트리 생성   
   SQL 문을 트리 구조(Parse Tree)로 변환
2. Syntax 체크   
   문법적 오류 체크
3. Semantic 체크  
   의미상 오류 체크

---

## SQL Optimization

옵티마이저는 SQL을 가능한 여러 실행 경로를 비교한 후 가장 비용(Cost)이 낮은 실행 계획을 선택한다.

### 옵티마이저가 고려하는 요소

- Index 사용 여부
- Join 방식
- Table Access 방식
- Predicate Pushdown
- Sort 여부

같은 SQL이라도 실행 방법이 여러 개 존재한다.  
Full Table Scan, Index Scan, Index Range Scan 등 이중 옵티마이저는 통계정보를 기반으로 비용을 계산하여 선택.

---

## 로우 소스 생성

- 옵티마이저가 선택한 경로를 실행 가능한 형태(프로시저)로 포맷팅 한다.

이를 Row Source Tree 라고 한다.

예시

```
SELECT
 └── NESTED LOOP
      ├── INDEX RANGE SCAN
      └── TABLE ACCESS
```

---

## 실행 계획 (Execution Plan)

실행 계획은 SQL을 처리하는 단계별 절차.

### 대표적인 실행 방식

- Table Full Scan
- Index Scan
- Nested Loop Join
- Hash Join
- Sort Merge Join

실행 계획은 `EXPLAIN` 또는 `EXPLAIN ANALYZE` 명령어로 확인할 수 있다.

## SQL 공유 및 재사용

DB는 실행 계획을 'Library Cache' 에 저장한다.  
이 캐시에 저장된 실행 계획은 다른 SQL 실행 시 재사용될 수 있다.

---

## Hard Parsing vs Soft Parsing

### Hard Parsing

캐시에 SQL 실행계획이 없는 경우  
SQL → Parsing → Optimization → Execution Plan 생성  
비용이 크다, 느리다.

### Soft Parsing

이미 캐시에 실행 계획이 존재하는 경우  
SQL → Library Cache 조회 → Execution

---

## Prepared Statement

SQL을 미리 컴파일하여 실행 계획을 재사용하는 방식

예시

```sql
SELECT *
FROM customer
WHERE login_id = ?
```

실행시 ? 에 값 바인딩 되어

```sql
SELECT *
FROM customer
WHERE login_id = 'jinho'
SELECT *
FROM customer
WHERE login_id = 'kim'
```

같은 실행 계획을 재사용한다.
