# PostgreSQL

## Architecture

### 동작 과정

```
Client
   │    클라이언트가 연결 요청
   ▼
Postmaster
   │    Postmaster가 backend process 생성
   ▼
Backend Process
   │    이후 클라이언트는 backend process와 직접 통신
   ▼
Database Files
```

## Query Processing Architecture

```
SQL Query 
   ↓
Parser // SQL 문법 분석
   ↓
Rewrite System // Rule 시스템 적용
   ↓
Planner / Optimizer
   ↓
Executor
   ↓
Storage Engine
```

PostgreSQL은 **Heap Table + Index 구조**를 사용하는 관계형 데이터베이스이다.  
테이블 데이터는 **heap 파일**에 저장되고, 인덱스는 해당 row의 위치를 가리키는 **TID(tuple id)** 를 저장한다.
(InnoDB처럼 테이블 자체가 인덱스 기준으로 정렬 저장되지 않는다.)

[vs InnoDB](../MySQL/수정중.md#Architecture)

### InnoDB

- 기본 키(Primary Key) 기준으로 테이블 데이터 자체가 B+Tree 리프 페이지에 저장된다. 때문에 PK 조회에 유리하다. (clustered index기반)
- MVCC

### PostgreSQL

- 테이블은 heap에 저장되고, 인덱스는 해당 row 위치를 가리키는 'tuple id'를 들고 있다.  
  즉, PostgreSQL은 MySQL InnoDB처럼 테이블 그 자체가 clustered 되어 있지 않다.

---

## TOAST 저장 전략

PLAIN X X 압축이나 외부 저장을 하지 않습니다. 크기가 작은 고정 길이 타입(Integer 등)에 사용됩니다. EXTENDED O O 기본값. 먼저 압축을 시도하고, 그래도 크면 별도의 TOAST 테이블로 옮깁니다. EXTERNAL X O 압축은 하지 않고, 큰 데이터만 외부 TOAST 테이블로 옮깁니다. (텍스트 검색 등에 유리)
MAIN O X 압축을 시도하지만, 가능한 한 메인 테이블에 남기려 노력합니다. (최후의 수단으로만 외부 저장)