# I/O 매커니즘

데이터베이스 성능의 대부분은 I/O에서 결정된다.  
특히 디스크 접근은 메모리 접근보다 매우 느리기 때문에 DB 성능 튜닝의 핵심은 디스크 I/O를 최소화하는 것이다.

## Buffer Cache

Buffer Cache는 디스크 블록을 메모리에 캐싱하는 공간이다.

---

## Logical I/O vs Physical I/O

### Logical I/O

Buffer Cache에서 블록을 읽는다. 때문에 디스크 접근이 발생하지 않는다.

### Physical I/O

디스크에서 블록을 읽는 것. Buffer Cache miss의 경우에 디스크에 접근하는 경우다.

---

## Random I/O

디스크의 임의 위치에서 데이터를 읽는 방식

### Random I/O  특징

- 디스크 헤드 이동 발생, 매우 느리다.
- 대표적으로 Index Scan에서 발생

## Sequential I/O

연속된 블록을 순차적으로 읽는 방식

- 디스크 헤드 이동 최소화
- 대표적으로 Table Full Scan에서 발생

---

DB 성능 튜닝의 핵심 원칙은 Physical I/O 최소화 이다. 이를 위해

- Index 사용
- Buffer Cache 활용
- Query 최적화
- Access Path 최적화

의 방법을 사용한다.