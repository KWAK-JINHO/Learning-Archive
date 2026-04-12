# Singleton Pattern
특정 클래스의 인스턴스가 오직 하나만 생성되도록 보장하고,  
그 인스턴스에 전역적으로 접근할 수 있게 하는 패턴

### 기본 설계 및 구현
- 생성자를 private로 선언하여 외부에서 직접 인스턴스를 생성할 수 없게 한다.
- static 메서드를 통해 인스턴스에 접근가능하게 한다.

```markdown
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

## new vs Singleton Pattern

### new 키워드
- 매번 새로운 객체를 생성
- 메모리에 별도의 인스턴스 생성(독립적인 상태)

### Singleton
- 항상 동일한 객체 반환
- 처음 호출될 때만 새 객체를 생성한 후, 메모리에 단 하나의 인스턴스만 존재(같은 객체의 상태를 공유)
