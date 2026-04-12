# 개요

JVM(Java Virtual Machine)은 Java 바이트코드(.class 파일)를 실행하기 위한 가상 엔진 OS에 독립적인 실행환경을 제공(OS로부터 메모리를 할당)하며 GC(Garbage Collection)를 제공해 메모리 관리를 수행한다.

# 내부구조

## Class Loader

바이트코드(.class)를 OS에 할당받은 메모리(Runtime Data Area)에 로딩하고 링킹하는 역할

- 로드: .class 파일을 메모리의 Method Area에 로드
- 링킹: .class 파일을 검증

## Runtime Data Area

- OS로부터 할당받은 메모리 영역
- Method Area, Heap, Stack , PC Register, Native Method Stack으로 구성

### 공유 영역

- Method area: 클래스, 필드, 메서드, 인터페이스, 클래스변수 등을 보관
- Heap:  new키워드로 생성된 객체의 배열, 인스턴스 변수를 보관. GC가 Heap에서 참조되지 않는 객체를 확인하고 제거한다.

### 스레드 개별 영역(Thread-Safe)

- Stack: 메서드의 호출정보(매개변수, 지역변수, return 값, 연산 중 생기는 값등)저장. 메서드 수행이 끝나면 삭제
- PC Register: 스레드별 현재 실행 중인 명령어 주소 기록
- Native method stack: C/C++ 등 네이티브 코드 실행용 스택

## Execution Engine

- Interpreter: 바이트코드를 한 줄씩 해석
- JIT(Just-In-Time) 컴파일러: 반복되는 코드(Hot Spot)을 기계어로 컴파일하여 캐싱
- GC(Garbage Collector): Heap영역의 미사용 객체를 자동 정리 [[ GC 추가 내용은 여기를 참고 ]](./gc)
- JNI(Java Native method Interface): OS 고유 기능 및 외부 언어 라이브러리와의 통로

# 런타임 생명주기

1. 빌드: javac(컴파일러)이 소스(.java)를 바이트코드(.class)로 변환.
2. 프로세스 생성: OS가 JVM에 메모리를 할당하며 엔진 구동
3. 동적 스레드 확장: 런타임 중 사용자 요청 발생시 JVM이 OS에 System Call을 보내 커널 스레드를 생성, 자바 스레드와 1:1 매핑
4. 자원 할당: 새로 생성된 스레드만을 위한 Stack과 PC Register를 즉시 동적 할당