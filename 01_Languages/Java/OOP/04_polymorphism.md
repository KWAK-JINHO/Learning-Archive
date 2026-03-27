# 다형성

## 다형성이란?

하나의 객체가 여러가지 타입을 가질 수 있는 능력으로 기능이 계속 추가될 가능성이 있는 로직에 사용.

예시) 결제(네이버페이, 카카오페이, 삼성페이 등..), 로그인(카카오, 구글, 애플 등등..), 알림(핸드폰, 이메일 등등..)

## 코드예시

```java
// 모든 알림이 지켜야할 약속을 만든다.
public interface Notification {
    void send(String message); // 알림을 보내는 행위를 정의
}

// 각 알림(이후에 추가될 알림들 또한) 수단은 자기만의 방식으로 기능을 구현한다.
public class EmailNotification implements Notification {
    @Override
    public void send(String message) {
        System.out.println("이메일 발송: " + message);
    }
}

public class SmsNotification implements Notification {
    @Override
    public void send(String message) {
        System.out.println("SMS 발송: " + message);
    }
}

// 사용
public class NotificationService {
    // Notification 인터페이스를 구현한 어떤 객체든 매개변수로 받을 수 있다.(다형적 매개변수)
    public void sendAlert(Notification notification, String msg) {
        // 동적 바인딩
        notification.send(msg);
    }
}
```

이는 수정에는 닫혀있고 확장에는 열려있는(OCP) 구조를 설계할 수 있다.

## instanceof

`if (참조변수 instanceof 타입) {}`

"왼쪽(참조변수) 변수를 오른쪽(타입) 으로 사용할 수 있는가?" 런타임에 실제타입을 확인(형변환이 가능한가?)하는 연산자.

업캐스팅(자식 -> 부모)은 자유롭지만 다운캐스팅은 메모리에 실제 자식 객체가 들어있을 때만 가능하다. 이를 안전하게 instanceof로 체크한다.ㄴ

### 코드 예시

```java
class 새 {
    void 날다() {
        System.out.println("날아갑니다.");
    }
}

class 타조 extends 새 {
    @Override
    void 날다() {
        throw new UnsupportedOperationException("타조는 못 날아요");
    }
}

public class 동물원 {
    public void move(새 bird) {
        if (bird instanceof 타조) {
            System.out.println("타조는 뛰어갑니다.");
        } else {
            bird.fly();
        }
    }
}
```

instanceof 를 많이 사용하는(= LSP를 지키지 못한) 경우는 잘못된 설계이거나 외부 라이브러리를 사용할 때이다.

```java
interface Movable {
    void move();
}

class 새 implements Movable {
    @Override
    public void move() {
        System.out.println("날아갑니다.");
    }
}

class 타조 implements Movable {
    @Override
    public void move() {
        System.out.println("뛰어갑니다.");
    }
}

public class 동물원 {
    public void moveAnimal(Movable animal) {
        animal.move();
    }
}
```

## 요약

컴파일 타임에 타입을 확정하지 않고 추상적인 인테페이스에 의존함으로써 런타임에 유연한 동작을 하게 만들 수 있다.