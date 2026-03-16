# JPA란?

자바 ORM(Object RelationShip Mappping: 객체와 관계형 DB 매핑)기술에 대한 API 표준 명세.

개발자가 CRUD문을 작성할 때 자바 객체를 애플리케이션과 DB 사이에서 넣고 뺄 때 매핑을 반복해야 하는 문제가 발생 -> 객체는 OOP적으로, DB는 DB대로 설계 후 ORM이 중간에서 2개를 매핑하는 역할.

JPA는 ORM기술의 표준 명세로 인터페이스이고, 이를 구현한 대표적인 구현체인 Hibernate를 사용

# JPA 작동방식

- 애플리케이션 실행
- EntityManagerFactory 생성  (애플리케이션 시작 시 한번만 생성된다.)
- EntityManager생성 (트랜잭션 단위로 생성 후 닫는다.)
- persist(): 1차 캐시에 저장. ( 아직 데이터베이스에는 저장되지 않은 상태, 쓰기 지연 적용)
    - 쓰기 지연: JPA는 쓰기 지연 SQL 저장소에 커밋전까지 SQL를 저장했다가 Tx 커밋 시점에 한번에 실행
- 트랜잭션을 `commit`하면 영속성 컨텍스트에 있는 엔티티가 데이터베이스에 반영됨.
- find(): 1차 캐시에서 바로 조회하고, 없으면 DB에서 가져온다.
- DB에서 가져온 엔티티는 PC에서 관리대상이 된다.(영속 상태)
- 변경 감지(더티 체킹): commit() 시 변경된 엔티티는 자동으로 UPDATE 실행
- 엔티티들은 변경감지 되는 상태이므로 set 으로 변경하면 JPA가 자동 감지함
- remove(): 삭제 후 commit() 시 DELETE 반영
- commit(): 쓰기 지연 SQL실행, DB반영 후 트랜잭션 종료
- EntityManager.close()

### flush() vs save()

- flush()의 경우 영속성 컨텍스트에(1차캐시)에 있는 변경 사항을 DB에 즉시 반영하지만, 트랜잭션이 롤벡되면 다시 취소된다.
- save()의 경우에는 새로운 엔티티이면 persist()를 호출, 있던 엔티티이면 merger()를 호출하며 트랜잭션이 커밋하는 시점에 자동으로 flush되어 DB에 반영된다.

### 커넥션 풀

-EntityManager는 생성될 때 커넥션을 바로 가져오지 않는다. -실제 데이터베이스 작업이 필요할 때 커넥션을 요청. -persist(), find(), merge(), remove() 등의 작업이 호출되면 커넥션 풀에서 커넥션을 가져온다. -작업이 끝나면 커넥션은 커넥션 풀로 반환 -트랜잭션이 없는경우 작업이 끝나는 즉시 반환

## 영속성 컨택스트

-Entity Manager에 의해 관리되며 JPA의 모든 동작이 이 안에서 이루어진다. -Map으로 되어 있다. -Key는 엔티티의 PK, Value에 엔티티 객체 자체가 저장된다.

### 영속성 컨텍스트의 특징

영속성 컨텍스트는 엔티티를 식별자 값으로 구분한다. 때문에 영속 상태는 식별자 값이 반드시 있어야 한다.(없을 경우 예외발생)

### 영속성 컨텍스트 사용시 장점

- 1차 캐시
- 동일성 보장
- 트랜잭션을 지원하는 쓰기 지연 (INSERT, UPDATE, DELETE가 발생하면 DB에 즉시 반영하는게 아니고 1차캐시(영속성 컨텍스트)에 쌓아둔다. 후에 트랜잭션이 커밋 되는 시점에 한번에 전송됨)
- 변경 감지
- 지연 로딩(엔티티를 실제로 필요할 때 조회하는 방식)

## 2차 캐시

영속성 컨텍스트의 1차 캐시를 보완하기 위해 제공되는 전역적인 캐시 1차 캐시는 EntityManager 단위에서만 작동하지만 2차 캐시는 애플리케이션 범위에서 공유, 동일한 엔티티를 여러 EntityManager에서 재사용할 수 있도록 한다.

Hibernate에서는 @cache 어노테이션으로 지원한다.

읽기전용, 읽기-쓰기(데이터 변경시 캐시 갱신), 비관적 잠금(데이터 변경시 캐시 무효화, 갱신은 안한다.)

장점: 성능 향상, 트래픽 감소 단점: 데이터가 자주 변하면 일관성 유지 어려움, 캐시가 커지면 메모리 사용량 증가

# DAO(data accsess object) vs Repository

## DAO

SQL 최적화가 중요한 경우에 사용 ORM 사용하지 않는 경우 사용 레거시 시스템 -> 기존의 코드가 이미 사용중일 때

## Repository

도메인 중심의 설계(DDD)
ORM 기술(Hibernate)에 의존적이다. SQL추상화, 객체(Entity) 기반 작업 -> 쿼리메서드, JPQL, Querydsl 빠른 개발, 유지보수에 용이

#### 두 개념은 통합 사용이 가능하다. Repository를 통해 객체 중심의 데이터 접근을 처리, DAO를 통한 복잡한 SQL쿼리 작성

# JPA가 지원하는 쿼리

![[스크린샷 2024-12-05 오후 12.42.26.png]]

## 쿼리 메서드

JpaRepository<Post, Long>를 상속하여 사용하며, 메서드 이름을 통해 간단하게 쿼리를 작성할 수 있다. JPA 구현체인 Hibernate가 자동으로 생성해준다.

### 특징

- 간단한 조회 기능에 적합
- 메서드 이름이 길어질 수 있다.
- 복잡한 쿼리 작성은 어려움

## JPQL

객체지향 쿼리 언어로, 엔티티 객체와 매핑된 속성을 기준으로 SQL과 유사한 쿼리를 작성 SQL과 유사하지만 DB테이블이 아니라 엔티티를 대상으로 작업한다.(JPQL은 데이터 베이스 테이블을 전혀 알지 못한다.)

### 특징

- SQL과 비슷한 구문으로 사용하기 쉽다.
- 복잡한 쿼리 작성 가능
- 엔티티에 의존하므로 변경 사항 반영이 쉬움

### 사용예시

```
@Query("SELECT p FROM Post p WHERE p.title = :title")
List<Post> findPostsByTitle(@Param("title") String title);
```

## Querydsl

타입 세이프한 동적 쿼리를 작성할 수 있는 도구이다. 쿼리를 코드로 작성하며, 컴파일시점에 문법 오류 체크가 가능하다. 복잡한 동적 쿼리를 작성할 때 적합. 단점이 없다.

### 사용법

```
public List<Post> findPosts(String author, String category) {
    QPost post = QPost.post;

    BooleanBuilder builder = new BooleanBuilder();
    if (author != null) {
        builder.and(post.author.eq(author));
    }
    if (category != null) {
        builder.and(post.category.eq(category));
    }

    return queryFactory
        .selectFrom(post)
        .where(builder)
        .fetch();
}
```

# 엔티티와 데이터베이스 테이블 매핑

## @Entity

클래스에 선언하여 JPA가 관리하는 객체로 등록한다.

### 사용법

- 기본생성자가 반드시 필요
- final, enum, interface 엔티티로 사용할 수 없다.
- @Id(PK)가 반드시 선언되어야 한다.

## @Id

- 엔티티의 기본 키를 지정할 때 사용
- DB테이블의 PK에 매핑
- 주로 Long, Integer, UUID사용

## @GeneratedValue

기본 키의 값을 자동으로 생성하도록 설정

### 사용법

속성을 통해 전략을 선택 가능

- `IDENTITY`: 데이터베이스에 위임 (MySQL의 AUTO_INCREMENT 등)
- `SEQUENCE`: 데이터베이스 시퀀스 사용 (주로 Oracle, PostgreSQL)
- `TABLE`: 키 생성용 테이블을 사용
- `AUTO`: 데이터베이스에 맞게 자동 선택

## @Column

엔티티의 필드를 테이블의 컬럼에 매핑할 때 사용

### 사용법

- `name`: 테이블에서 사용할 컬럼 이름
- `nullable`: `null` 허용 여부 (기본값은 `true`)
- `length`: 문자 길이 지정 (기본값은 255)
- `unique`: 고유 키 설정

```
예시)
@Column(name = "user_name", nullable = false, length = 50)
private String name;
```

## 기본 생성자와 Getter/Setter

- JPA는 프록시 객체를 생성하거나 DB에서 조회된 데이터를 바탕으로 엔티티를 생성할 때 리플렉션을 사용하고 리플렉션은 기본생성자를 통해 객체를 생성
- 따라서 매개변수가 없는 public 또는 protected 기본 생성자가 필요하다.

## @Table(선택사항)

엔티티가 매핑될 테이블 이름이나 세부정보 지정

### 주요 속성

- `name`: 테이블 이름
- `schema`: 스키마 이름
- `uniqueConstraints`: 유니크 제약 조건 예시)

```
@Entity 
@Table(name = "users") 
public class User {
	...
}
```

# 연관관계 매핑

## 연관관계 종류

- 단방향 매핑: 한 엔티티에서 다른 엔티티를 참조하지만, 반대 방향에서는 참조하지 않는 관계
- 양방향 매핑: 서로 상대방 엔티티를 참조하여, 객체 그래프 탐색이 가능한 관계

## 주요관계

### **일대일(@OneToOne)**: 두 엔티티가 1:1로 연결

### **일대다(@OneToMany)**: 하나의 엔티티가 여러 엔티티와 연결

- 예시) 사용자가 여러 개의 주문을 가질 수 있음.

```
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @OneToMany(mappedBy = "user") // 연관관계의 주인은 Order의 user 필드
    private List<Order> orders = new ArrayList<>();
}

```

mappedBy

- 연관관계의 주인을 지정.
- 주인은 실제 외래 키를 관리하는 엔티티.

### **다대일(@ManyToOne)**: 여러 엔티티가 하나의 엔티티와 연결

- 예시) 주문(`Order`)은 하나의 사용자(`User`)와 연결됨.

```
@Entity
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne
    @JoinColumn(name = "user_id") // 외래 키 컬럼 지정
    private User user;
}

```

### 다대다

관계형 데이터베이스는 정규화된 테이블 2개로 다대다 관계를 표현할 수 없다. 때문에 보통 다대다 관계사이에 연결 테이블을 추가하여 이를 해소한다.

## 상속관계 매핑

슈퍼타입, 서브타입의 관계로 나타낸다.

1. 조인전략: 부모테이블의 기본 키를 받아 기본 키 + 외래 키를 사용하는 여러 엔티티가 있다. 장점

- 테이블이 정규화된다.
- 외래 키 참조 무결성 제약조건을 활용할 수 있다.
- 저장공간의 효율화 단점
- 조인이 잦아 성능 저하
- 조회 쿼리 복잡

2. 테이블 하나에 모두 합치기 장점

- 조회가 빠르다.
- 쿼리가 단순하다 단점
- 자식 엔티티가 매핑한 컬럼은 모두 null 허용
- 테이블이 비대해진다.(이는 상황에 따라 느려진다는 말과 동일하다.)

## @MappedSuperClass

단순하게 코드의 중복을 없애기 위해서 공통되는 필드값을 따로 빼주어 상속받아 사용하는 전략. 상속관계와다르게 부모테이블을 생성할 수도 없고 부모 엔티티를 직접 조회할 수 없다.

## 식별 관계, 비식별 관계와 복합키에 대해서

식별관계는 부모 테이블의 기본 키를 내려받아서 자식 테이블의 기본 키 + 외래 키로 사용하는 관계이고 비식별관계는 부모 테이블의 기본 키를 자식 테이블의 FK로만 사용하고, 기본 키를 따로 가지는 관계입니다.

식별관계는 복합 키를 위한 식별자 클래스를 만들어야 하므로 비식별관계를 사용하는것이 편리하다. (부모의 PK를 받는 것이 일반적이지만, 단일 키를 별도로 추가하는 방식도 있다.)

📌 복합키에 대해서 복합키의 사용여부에 대해서는 찬반의견이 갈리는 것 같다. 내 생각으로는 적어도 JPA에서는 아래 두가지 이유 때문에 단일 키를 사용하는게 좋은 방법인것 같다.

- 코드가 너무 복잡해진다.
    - @IdClass 또는 @EmbeddedId 추가 설정 및 equals()와 hashCode()필수 구현
- 성능 저하
    - 인덱스가 여러 컬럼에 걸쳐 생성되어 검색속도가 느려질 수 있다. 또한 조인할 때 복합키의 모든 필드를 비교해야 하기 때문에 부담을 준다. JPA가 아니더라도 복합키를 사용하면 추후에 새로운 서비스가 생길 때 확장성 측면에서 어려움을 겪을 확률이 높다고 생각되어진다.

복합키의 장점으로는 데이터 무결성 보장이라는 아주 큰 장점이 있지만 JPA는 PK + @UniqueConstraint 로 해결이 가능하다.

# 프록시

실제 엔티티를 대신하는 가짜 객체라고 생각하면 된다. JPA는 엔티티를 조회할 떄 프록시 객체를 생성, 프록시 객체는 실제 엔티티를 대신하여 동작하며, 필요한 시점에 DB에서 데이터를 가져와 채운다.

📌 보통 지연로딩을 위해 프록시 객체를 사용하는 것이 기본설정이며, 즉시 로딩 설정이 되어 있거나 프록시 생성 비활성화하면 프록시 객체를 생성하지 않는다.

## 프록시 객체가 필요한이유

- 지연로딩(Lazy Loading): 모든 연관된 엔티티를 한꺼번에 조회하는 것이 아니라 필요한 시점에 DB에서 조회하여 성능을 향상시키기 위해 사용
- 프록시 객체는 실제 엔티티를 상속하기 때문에 실제 엔티티처럼 사용할 수 있다. -> 성능, 메모리 측면에서 좋으나 초기화 시점을 알기 어렵고 지연 로딩 시 예외가 발생할 수 있어 조심행함
- 사용예시) @ManyToOne(fetch = FetchType.LAZY)

# OSIV

OSIV(Open Session In View) 패턴의 기본 개념 OSIV 패턴은 데이터베이스 세션을 HTTP 요청의 시작부터 끝까지 열어 두는 방식을 말한다.

	지연로딩이란?
	엔티티가 처음 조회될 때 즉시 모든 연관된 엔티티를 가져오는 것이 아니라, 실제 해당 데이터가 필요할 때 가져오는 방식
	
	how is that possible?
	프록시 객체의 활용
	JPA는 연관된 엔티티를 즉시 로드하는 것이 아니라 프록시 객체로 감싸서 반환한다.
	프록시 객체는 실제 데이터는 없고 특정 필드에 접근할 때 쿼리를 실행하여 데이터를 가져온다.
	
	what problem occurs by lazy loading?
	N+1 문제 발생 가능 
	부모 엔티티를 로드한 후 그에 관련된 각자식 엔티티마다 별도의 데이터베이스 쿼리를 실행하게 되 면 총 1번(부모) + N번(각 자식) 쿼리가 발생하는 성능상의 이슈

이 패턴의 주된 목적은 지연로딩 문제를 해결하기 위함이지만 성능 저하를 일으 킬 수 있기 때문에 장단점을 잘 알아야 한다.

이 패턴은 DB 커넥션을 오래 점유하게 되어, 고성능을 요구하는 애플리케이션에서는 문제가 될 수 있다.

'OpenEntityManagerInViewInterceptor' 또는 'OpenSessionInViewFilter'를 사용하여 구현

데이터베이스 커넥션을 오래 점유할 수 있으므로, 커넥션 풀 설정에 주의해야 합니다.