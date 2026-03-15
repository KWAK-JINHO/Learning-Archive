# Singleton Pattern

"하나의 인스턴스만 있도록 하기"

### 싱클톤 패턴이란?

- 클래스의 인스턴스가 오직 하나만 생성되도록 보장하는 디자인 패턴

### 싱글톤 패턴의 목적

- 어플리케이션 전체에서 하나의 공유 인스턴스만 사용하고자 할 때 사용한다.

### 기본 설계 및 구현

- 생성자를 private로 선언하여 외부에서 직접 인스턴스를 생성할 수 없게 한다.
- static 메서드를 통해 인스턴스에 접근가능하게 한다.

```commandline
public class Singleton {
    // 유일한 인스턴스를 저장할 정적 변수
    private static Singleton instance;

    // private 생성자로 외부에서 인스턴스 생성을 막음
    private Singleton() {}

    // 인스턴스를 얻는 public 정적 메소드
    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }

    public void showMessage() {
        System.out.println("안녕하세요, 저는 싱글톤 인스턴스입니다!");
    }
}
```

```commandline
public class Main {
    public static void main(String[] args) {
        Singleton singleton = Singleton.getInstance();
        singleton.showMessage();
    }
}
```

### new vs Singleton Pattern

- new
    - 매번 새로운 객체가 생성
    - 메모리에 별도의 인스턴스가 생성
    - 각 객체는 독립적인 상태

- getInstance()
    - 항상 동일한 객체 반환
    - 처음 호출될 때만 새 객체를 생성한 후, 메모리에 단 하나의 인스턴스만 존재
    - 모든 호출에서 같은 객체의 상태를 공유

### 조심해야할 점

- 전역상태를 만들어 객체간 결합도를 높인다는게 특징이다.
- 캡슐화와 모듈화를 해칠 수 있어 객체지향적으로 좋지는 않다.
- 따라서 설정관리, 로깅등 에서만 신중히 사용권장
