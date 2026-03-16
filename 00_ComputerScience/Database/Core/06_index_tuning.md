# Index Tuning

## 목차

---

## 커버링 인덱스

쿼리에 필요한 모든 컬럼이 인덱스에 포함되어 있어  
**테이블 접근 없이 인덱스만으로 결과를 반환할 수 있는 경우**를 말한다.

예시

```sql
SELECT name, age
FROM users
WHERE name = 'kim';
```

인덱스 (name, age) 경우 DB는 테이블을 접근하지 않고 인덱스만 사용하여 결과를 반환

## 인덱스 리빌드

대량의 INSERT / UPDATE / DELETE가 발생하면  
인덱스 내부에 **Fragmentation(단편화)** 이 발생할 수 있다.

이 경우 인덱스를 재구성(Rebuild)하면 디스크 공간과 탐색 성능을 개선할 수 있다.