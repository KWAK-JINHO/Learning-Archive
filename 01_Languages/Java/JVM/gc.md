# Garbage Collector

JVM은 "이 객체가 유효한가?"를 판단하기 위해 Root Set 부터 참조사슬을 따라간다. 어떤 경로로도 Root Set과 연결되지 않은 객체가 GC의 대상이된다.

## GC의 기본 알고리즘: Mark-and-Sweep

- Mark: Root Set에서 참조 사슬이 연결된 객체를 찾아 마킹한다.
- Sweep: 마킹되지 않은 객체들을 Heap에서 제거한다.
- Compact: 제거 후 메모리 단편화(Fragmentation)을 해결하기 위해 빈 공간을 채운다.(선택적)

## GC의 관리 전략

객체는 대부분 금방 소멸한다는 가정하에 Heap을 나눈다.

- Young Generation: 새로운 객체가 생성되는 곳. 여기서 발생하는 GC를 Minor GC라고 한다. Eden, S0, S1 3가지 영역으로 나뉜다.
- Old Generation: 객체가 오랫동안 사용되어 Minor GC에서 살아남은 객체가 이동하는 곳. 여기서 발생하는 GC는 Full GC라고 한다.

## GC 튜닝

GC튜닝의 핵심은 Stop-the-World(GC를 실행할 때 모든 스레드가 멈추는 현상)를 최소화하여 Latency를 줄이고 Throughput을 높이는 것

### 주요 GC 알고리즘

- Parallel GC: Java8의 기본 GC. 여러 개의 스레드로 GC를 처리하여 throughput을 높지만 STW 시간이 다소 길다.
- G1GC: Java9 이후 표준. Heap을 Region으로 나누어 Garbage가 많은 영역부터 우선순위로 청소한다. STW 시간을 예측할 수 있다.
- ZGC: Java15 이후 도입. 대용량 Heap 에서도 STW 시간을 10ms 이하로 유지하는 초저지연 GC.

### Java GC 최적화 전략

1. Heap 크기 최적화

-Xms 와 -Xmx를 동일하게 설정하는 것을 권장. Heap 크기가 동적으로 변한다면 OS 메모리 재할당 오버헤드가 발생하기 때문이다.

- -Xms (Initial Heap Size): JVM이 시작될 때 할당하는 최소 메모리 크기
- -Xmx (Maximum Heap Size): JVM이 사용할 수 있는 최대 메모리 크기

2. 객체 생성 최적화

- String 결합에 `+` 연산 지양. StringBuilder를 사용해 불필요한 객체 생성을 막아야 한다.
- 기본형 타입을 사용하면 스택에서 처리되거나 메모리 점유가 적다. Integer, Long 대신 int, long 사용하는것이 대량의 데이터를 처리할 때 유의미 할 수 있다.
- DB에서 대량의 데이터를 한꺼번에 조회하여 리스트에 담으면 이 객체들은 Eden영역을 순식간에 점유한다. Eden 영역이 꽉 차면 Minor GC가 발생하게 되는데
  이 횟수가 빈번해지면 application throughput 저하로 이어진다. 적절한 Pagination과 Stream 처리를 통해 메모리에 올라오는 객체 수를 제한해야 한다.

3. JVM 옵션

- `-XX:+UseG1GC`: G1GC사용
- `-XX:MaxGCPauseMillis=n`: 목표 최대 STW 시간 설정. 너무 짧게 설정할 경우 GC가 너무 자주 발생하여 throughput이 저하될 수 있다.
- `-XX:ParallelGCThreads=n`: GC를 병렬로 처리할 스레드 수 지정. CPU 코어 수에 맞춰 설정하는 것이 일반적
- `-XX:+PrintGCDetails`/`-Xlog:gc*`: GC 발생 상세 로그 출력