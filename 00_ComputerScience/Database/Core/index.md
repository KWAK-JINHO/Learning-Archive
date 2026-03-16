# 인덱스

인덱스는 **<Key, ROWID> 쌍을 정렬된 상태로 저장하는 데이터 구조**이다.  
DBMS는 인덱스를 통해 **검색 조건에 해당하는 Key를 빠르게 찾고**, 해당 Key에 연결된 **ROWID를 이용해 실제 데이터가 저장된 위치에 접근한다.**

대부분의 DBMS는 인덱스를 구현할 때 **[B+Tree](../../DataStructure/b_tree.md)** 구조를 사용한다.

데이터베이스 성능 튜닝의 핵심은 **디스크 I/O를 최소화하는 것**이며, 특히 **랜덤 I/O(Random I/O)를 줄이는 것**이 중요하다.

## ❓ 왜 DB 인덱스는 Binary Tree가 아니라 B+Tree를 사용할까

데이터베이스에서 인덱스는 **디스크 기반 구조**이다.

1. Binary Tree 보다 높이가 낮은 B-Tree는 그만큼 **탐색 시 디스크 I/O 횟수가 감소**.
2. DB에서는 다음과 같은 범위 기반 검색이 매우 자주 사용된다.
    ```sql
    WHERE age BETWEEN 20 AND 30
    ORDER BY age
    LIKE 'abc%'
    ```
   B+Tree는 **Leaf Node가 Linked List 형태로 연결**되어 있다.
3. **Sequential I/O 활용**  
   Leaf Node들이 서로 연결되어 있어 데이터를 **순차적으로 탐색(Sequential Scan)** 할 수 있다.  
   이는 디스크에서 **Sequential I/O**를 사용할 수 있게 하여 성능 향상에 유리하다.

## 인덱스 구조 및 탐색

DB 테이블에서 데이터를 찾는 방법은 크게 두 가지가 있다.

1. Full Table Scan
2. Index Scan

### 인덱스 탐색 과정

- 수직적 탐색 (Index Traversal)  
  인덱스 트리를 따라 내려가며 검색 시작 지점을 찾는 과정

- 수평적 탐색 (Index Scan)  
  Leaf Node에서 조건에 맞는 데이터를 찾는 과정

### 인덱스를 사용할 수 없는 조건

- 컬럼값을 변경시킬 때
   ```sql
   WHERE SUBSTR(생년월일, 5, 2) = '05'
   WHERE NVL(주문수량, 0) < 100
   ```

- 앞쪽에 와일드카드 사용
   ```sql
   WHERE 업체명 LIKE '%대한%'
   ```

- IN 조건을 사용할 때 (인덱스를 여러번 수행함)
   ```sql
   WHERE 전화번호 IN (:tel_no1, :tel_no2)
   ```
  각 값에 대해 인덱스 탐색이 여러번 수행되므로 경우에 따라 Full Scan이 효율적일 수 있다.

### 인덱스를 이용한 sort 연산 생략

인덱스는 Key 기준으로 이미 정렬된 상태를 유지한다.  
인덱스를 사용하면 `ORDER BY`를 생략할 수 있다.(Sort 연산 생략)

### 자동형변환

DB에서는 데이터 타입이 다른 경우 자동 형변환이 발생할 수 있다.

형변환 우선순위

```
(약함) 문자형(CHAR, VARCHAR) < 숫자형(INTEGER, DECIMAL) < 날짜/시간형(DATE, TIMESTAMP) (강함)
```

- 날짜 비교 시 명확한 날짜 포맷을 지정하지 않으면 의도하지 않은 형변환이 발생하여 인덱스를 사용할 수 없는 경우가 있다.

## 인덱스 종류

DB 인덱스는 데이터 저장 방식에 따라 두 가지로 구분된다.

### Clustered Index

Clustered Index는 테이블의 실제 데이터가 인덱스 Key 순서로 정렬되어 저장되는 인덱스이다.   
테이블당 하나만 존재하며, Leaf Node에 실제 데이터가 저장

### Non-Clustered Index

Non-Clustered Index는 인덱스와 실제 데이터가 분리되어 저장되는 인덱스이다.  
테이블당 여러개 생성 가능하며, Leaf Node에는 실제 데이터가 아니라 ROWID(또는 PK) 가 저장.  
DB는 인덱스를 통해 ROWID를 찾은 후 실제 데이터에 접근한다.

## 복합 인덱스

복합 인덱스는 두 개 이상의 컬럼을 조합하여 생성하는 인덱스.

복합 인덱스 생성하기

```sql
CREATE INDEX idx_user_name_age
    ON users (name, age);
```

### 특징

인덱스는 왼쪽 컬럼부터 정렬된다. 때문에 (name, age) 복합인덱스를

```sql
WHERE name = 'kim'
WHERE name = 'kim' AND age = 20
```

이경우에는 사용할 수 있지만

```sql
WHERE age = 20
```

이 조건에서는 사용할 수 없다.