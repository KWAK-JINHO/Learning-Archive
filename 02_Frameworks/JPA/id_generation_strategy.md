# JPA ID 생성 전략

JPA에서는 엔티티의 기본 키(PK)를 자동으로 생성하기 위해 `@GeneratedValue`를 사용한다.

```markdown
@Id @GeneratedValue(strategy = GenerationType.XXX)
private Long id;
```

## ID 생성 전략 종류

### 1. IDENTITY

- DB의 auto-increment 사용
- insert 시점에 PK 생성  
  JPA에서 엔티티를 영속하기 위해선 식별자가 필요한데, IDENTITY 전략에서는 이 식별자가 DB에 저장되어야 할당되기 때문에
- 배치 insert 비효율적
- MySQL, PostgreSQL 등에서 사용

### 2. SEQUENCE

- DB의 sequence 객체 사용
- insert 전에 ID 미리 조회
- batch insert 가능 (성능 좋음)
- allocationSize로 성능 최적화 가능
  ```markdown
  @SequenceGenerator(
  name = "user_seq",
  sequenceName = "user_seq",
  allocationSize = 50
  )
  ```
- Oracle, PostgreSQL 등에서 사용

### 3. TABLE

- 별도의 테이블을 만들어 ID 관리(`@TableGenerator`로 테이블 설정)
- DB와 한번더 통신하기 때문에 거의 사용하지 않는다.

### 4. AUTO

- JPA가 DB에 맞춰 자동 선택 (MySQL → IDENTITY, Oracle → SEQUENCE 등)