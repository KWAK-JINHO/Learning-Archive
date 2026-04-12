# JPA 내부동작에 대해서 알아보자

JPA는 EntityManager를 통해 영속성 컨텍스트를 조작

# EntityManager

EntityManger는 JPA에서 엔티티를 관리하는 핵심 인터페이스이다.

- 엔티티의 생성, 조회, 수정, 삭제를 담당
- 영속성 컨텍스트에 접근하는 유일한 창구
- JPA의 대부분 동작은 EntityManager를 통해 수행된다.

---

## EntityManager의 생명주기

1. EntityManagerFactory 생성
    - 애플리케이션 시작 시 1회 생성
    - 비용이 크기 때문에 재사용한다.

2. EntityManager 생성
    - 트랜잭션 단위로 생성
    - 스레드 간 공유 ❌ (thread-safe 아님)

3. 사용 후 종료
    - close() 호출로 종료

```markdown
EntityManager em = emf.createEntityManager(); 
em.close();
```

---

## 역할

### 엔티티 저장

`em.persist(entity);`

- 영속성 컨텍스트에 저장 (1차 캐시)
- DB에 즉시 반영 ❌ (쓰기 지연)

### 엔티티 조회

`em.find(Member.class, id);`

- 1차 캐시 먼저 조회
- 없으면 DB 조회 후 캐시에 저장

### 엔티티 수정 (Dirty Checking)

`member.setName("변경");`

- 별도 update 호출 없음
- 트랜잭션 commit 시 자동 반영

### 엔티티 삭제

`em.remove(entity);`

- commit 시 DELETE 실행

### flush

`em.flush();`

- 영속성 컨텍스트 → DB 즉시 반영
- 트랜잭션은 유지됨

---

## 영속성 컨텍스트와의 관계

EntityManager(인터페이스)는 내부적으로 영속성 컨텍스트(저장소)를 관리하며, 모든 엔티티 상태 변경은 이 컨텍스트에서 이루어진다.

## 트랜잭션과의 관계

EntityManager는 트랜잭션과 함께 동작한다.

```markdown
em.getTransaction().begin(); 
// 작업 
em.getTransaction().commit();
```

commit 시 발생

쓰기 지연 SQL 실행 변경 감지 반영 DB 동기화

## 커넥션과의 관계

- EntityManager 생성 시 커넥션을 바로 가져오지 않음
- 실제 DB 작업 시점에 커넥션 획득

### 동작 흐름

1. EntityManager 생성
2. persist / find 호출
3. 커넥션 풀에서 커넥션 획득
4. 작업 후 반환

---

## 특징

- 트랜잭션 단위로 사용
- thread-safe 하지 않음
- 1차 캐시 기반 동작
- 쓰기 지연 지원
- 변경 감지 자동 처리