자바 프로그램이 런타임 시점에 자기 자신이나 다른 객체의 메타데이터를 동적으로 조사하고 수정할 수 있는 기능

메타데이터란? 클래스, 인터페이스, 메소드, 필드 등에 대한 정보

즉 프로그램이 돌아가는 도중에 자기 자신의 내부 구조를 살펴보고, 필요에 따라 그 구조를 바꿀 수 있다는 뜻.

동적으로 로딩하는데 그때 힙 메모리에 생기는 Class 객체의 정보를 가지고 필요한 정보를 만들어주는게 리플렉션이다.

# Reflection

Reflection은 **런타임 시점에 클래스의 메타데이터를 조회하고 조작할 수 있는 기능**이다.  
컴파일 시점이 아닌 실행 중에 클래스의 정보(필드, 메서드, 생성자 등)에 접근할 수 있다.

---

## 특징

- 클래스 정보 조회 가능
- 메서드/필드/생성자 접근 및 실행 가능
- 런타임 동적 객체 생성 가능

---

## 필요한 이유

일반적인 Java 코드는 컴파일 시점에 타입이 고정된다.

```markdown
User user = new User();  
user. getName();
```

하지만 Reflection을 사용하면

- 클래스 이름만 알고 있어도 객체 생성 가능
- 어떤 메서드가 있는지 몰라도 실행 가능
- 유연한 프레임워크 구현 가능 과 같은 이점을 가진다.

---

## 핵심 클래스

### Class

#### Reflection의 시작점

`Class<?> clazz = User.class;`

또는

`Class<?> clazz = Class.forName("com.example.User");`

#### Constructor (객체 생성)

```java
Constructor<?> constructor = clazz.getConstructor();
Object instance = constructor.newInstance();
```

#### Field (필드 접근)

```markdown
dField("name");  
field.setAccessible(true);

field.set(instance, "jinho");  
Object value = field.get(instance);
```

### 동작 흐름

1. Class 객체 획득
2. Constructor / Method / Field 조회
3. 필요 시 접근 제어 해제 (private, protected 설정 필드나 메서드에 `setAccessible(true)` 메서드 사용)
4. 실행 (invoke, newInstance 등)

---

## 주의할 점

- private을 강제로 해제하는것은 설계의도를 무시하는 행위다.
- setAccessible(true)를 호출하면 JVM은 매번 Security Manger를 통해 동작이 허용되는지 체크한다. 이 과정은 일반적인 호출보다 훨씬 느리다.
- 리플렉션으로 의존성을 만들면 클래스 내부의 이름만 바꿔도 컴파일 에러없이 런타임에 프로그램이 터진다. 내부 구현은 언제든 바뀔 수 있어 개발자가 예측하지 못하는 상황을 만들 수 있다.