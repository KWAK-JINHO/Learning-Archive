# 블로킹 (Blocking) vs 논블로킹 (Non-blocking)

핵심은 호출된 함수가 제어권을 누구에게 주는가?

## 블로킹 (Blocking)
호출된 함수가 자신의 작업을 마칠 때까지 제어권을 가지고 있음  
호출자는 아무 작업도 하지 못하고 대기 상태가 됨

## 논블로킹 (Non-blocking)
호출된 함수가 즉시 제어권을 호출자에게 돌려줌  
호출자는 작업 완료 여부와 관계없이 자신의 다른 작업을 수행할 수 있음

## 동기/비동기와의 상관관계

[sync_async.md (동기와 비동기 개념)](./sync_async.md)

동기/비동기(관심사)와 블로킹/논블로킹(제어권)의 조합에 따라 4가지 실행 모델이 만들어질 수 있다.

1. Sync-Blocking
   - 호출자가 기다리고(Sync), 제어권을 뺏김(Blocking)
   - 예시: JDBC, 일반 메서드 호출

2. Sync-NonBlocking
   - 제어권을 돌려받아 다른 일을 할 수 있지만(NonBlocking), 계속 작업 완료를 물어봄(Sync/Polling)
   - 예시: Java의 Future.isDone()

3. Async-NonBlocking
   - 제어권을 돌려받아 내 일을 하고(NonBlocking), 완료 통보가 올 때까지 신경 쓰지 않음(Async)
   - 예시: Node.js, Spring WebFlux, AJAX 요청

4. Async-Blocking (의도치 않은 병목)
   - 완료 통보를 받기로 했지만(Async), 정작 제어권을 뺏겨서 아무것도 못 함(Blocking)
   - 예시: 비동기 라이브러리 내부에 블로킹 API(JDBC 등)가 포함된 경우
