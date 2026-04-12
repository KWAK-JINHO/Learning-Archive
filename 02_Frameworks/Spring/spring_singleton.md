# Spring Singleton(싱글톤 스코프)

- 스프링 컨테이너(ApplicationContext) 내에서 특정 Bean이 단 하나만 생성되어 관리되는 방식.
- 기본적으로 스프링의 모든 빈은 싱글톤 스코프를 가짐.

## GoF 싱글톤 패턴 vs Spring 싱글톤 빈
[참고: GoF 싱글톤 패턴](../../03_Architecture/DesignPattern/singleton_pattern.md)

### 구현 방식
- GoF 패턴: 클래스 내부에 private 생성자를 두고, static 메서드(getInstance)로 직접 객체 생성을 제어함.
- Spring Bean: 평범한 자바 클래스(POJO)를 사용하며, 스프링 컨테이너가 제어권을 가져가서(IoC) 스스로 객체를 딱 하나만 생성하고 주입함.

### 관리 범위 (Scope)
- GoF 패턴: 클래스 로더나 JVM 전체에서 단 하나만 존재함.
- Spring Bean: 스프링 컨테이너(ApplicationContext) 하나당 단 하나만 존재함. (컨테이너가 여러 개면 빈도 여러 개일 수 있음)

### 유연성 및 다형성
- GoF 패턴
  - private 생성자로 인해 상속이 어렵고 확장이 제한됨
  - static 기반 구조로 인해 테스트 시 Mock 객체로 대체하기 어려움
- Spring Bean: 인터페이스 구현 및 상속이 자유로우며, 의존성 주입(DI)을 통해 테스트용 가짜 객체를 쉽게 갈아 끼울 수 있음.

### 생명주기
- GoF 패턴
  - 객체 생성과 관리 책임이 개발자에게 있음
  - 생성 시점, 소멸 시점에 대한 표준화된 lifecycle 없음
  - 필요하면 직접 초기화/정리 로직 구현해야 함
- Spring Bean
  - 컨테이너가 생명주기를 관리
  - 생성 → 초기화(@PostConstruct) → 소멸(@PreDestroy)
  - 의존성 주입, AOP 등과 함께 lifecycle이 통합 관리됨

## Stateless
[static 키워드](./cv_iv_static.md)

싱글톤 객체는 여러 스레드가 동시에 접근하므로 상태를 저장하는 변수[(IV)](./cv_iv_static.md)를 사용하는 것에 매우 주의가 필요.
- 무상태(Stateless)로 설계해야 하며, 공유되지 않는 지역 변수나 파라미터를 사용하여 동시성 문제를 방지해야 함.
