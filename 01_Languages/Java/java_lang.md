# Java Language (java.lang)

`java.lang` 패키지는 자바 프로그램의 근간이 되는 클래스들을 포함하며, 별도의 `import` 없이 사용할 수 있는 유일한 패키지.

---

## Object 클래스: 동등성과 해시

### equals()와 hashCode()의 재정의

equals(Object)가 true를 반환하는 두 객체는 반드시 동일한 hashCode 값을 가져야 한다.  
hash기반 컬렉션들은 hashCode로 먼저 데이터가 들어갈 위치인 버킷을 찾고, 해당 버킷에서 equals로 최종 비교를 수행하기 때문.

### 타입 체크 방식: getClass() vs instanceof

- getClass()  
  `this.getClass() == o.getClass()`를 사용해 런타임 시점에 정확히 같은 클래스인지 비교.

- instanceof  
  다형성을 사용하기 위해 사용. 상속 관계의 자식 클래스, JPA 프록시 객체까지 인정.

- `instanceof` 패턴 매칭이나 `switch` 패턴 매칭을 자주 사용
  ```java
  if (o instanceof Subscribe sub) {
      return Objects.equals(email, sub.email) && Objects.equals(category, sub.category);
  }

---

## 문자열 결합 도구

- Java 8에 도입
- String.join(): 단순 결합용. 가독성이 좋음.
- StringJoiner: 접두사/접미사가 필요하거나 동적 결합이 필요할 때 사용.

### 주의사항

- 성능상 `+` 연산보다 우수 (내부적으로 StringBuilder 활용)
- 대량의 데이터 처리 시에는 Stream API의 joining()이 더 유연함.
- Java 21 환경에서는 코드의 간결성을 위해 가급적 Stream 지향.

---