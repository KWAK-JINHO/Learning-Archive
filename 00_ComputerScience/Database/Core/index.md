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

# 인덱스 구조 및 탐색

DB 테이블에서 데이터를 찾는 방법은 크게 두 가지가 있다.

1. Full Table Scan
2. Index Scan

## 인덱스 구조

- 수직적 탐색: 인덱스 스캔 시작지점 찾는 과정
- 수평적 탐색: 데이터를 찾는 과정

* 인덱스를 가공하면 정상적으로 스캔 시작점을 찾을 수 없기 때문에 Range Scan이 불가해진다.

### 인덱스를 정상적으로 사용불가한 조건절

- WHERE Substr(생년월일, 5, 2) = '05' // 컬럼값 변환시킴
- WHERE nvl(주문수량, 0) < 100 // 컬럼값 변환시킴
- WHERE 업체명 like '%대한%' // 와일드카드가 앞에 있어서
- WHERE (전화번호 = :tel_no AND 고객명 = cust_nm) // 각 조건에 따라 별도 인덱스 사용
- WHERE 전화번호 in(:tel_no1, :tel_no2) // 전화번호 컬럼 인덱스 여부 따져야 한다. 각 값에 대해 인덱스 수행해야 한다. Full Scan이 효율적일 수도 있다.

## 인덱스를 이용한 sort 연산 생략

- 전체 테이블 스캔 n
- 인덱스 사용 logn
- 인덱스를 사용하면 정렬된 상태 반환. SORT ORDER BY 연산x

## 자동형변환

(약함) 문자형(CHAR, VARCHAR) < 숫자형(INTEGER, DECIMAL) < 날짜/시간형(DATE, TIMESTAMP) (강함)

- 날짜 포맷을 정확히 지정해주어야 한다.