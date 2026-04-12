# Filter vs Interceptor vs AOP

- "공통 관심사(Cross-Cutting Concerns) 분리" 목적.
- 중복 코드 제거로 재사용성 및 유지보수성 향상.

## Filter
- "서블릿 컨테이너" 레벨에서 동작함.
- DispatcherServlet 도달 전후에 실행됨.
- HTTP 요청 및 응답 데이터의 "전체적인 필터링" 담당.

## Interceptor
- "스프링 컨테이너" 내부에서 동작함.
- DispatcherServlet과 컨트롤러 사이에서 실행됨.
- "스프링 빈"에 자유롭게 접근 가능함.

## AOP
- "메서드 단위"로 동작
- 프록시 패턴을 사용하여 핵심 기능 외의 부가 기능을 동적으로 추가
- 비즈니스 계층의 공통 로직 처리에 특화

---

## 비교

| Feature | Filter | Interceptor | AOP |
| :--- | :--- | :--- | :--- |
| Container | Servlet Container | Spring Container | Spring Container |
| Execution Point | Request <-> Servlet | Servlet <-> Controller | Proxy <-> Method |
| Target | Web Application | Spring MVC | Methods (Service) |
| Request/Response | 조작 가능 | 참조 및 값 변경 | 파라미터/리턴값 조작 |
| Exception | Servlet level | @ControllerAdvice | @ControllerAdvice |

---

## 사용사례

### Filter
- 보안 및 인증: IP 차단, 전체 인증 체크.
- 인코딩: UTF-8 전역 설정.
- 로깅: HTTP 요청 원본 기록.

### Interceptor
- 세션 체크: 로그인 여부 확인.
- 권한 제어: 세부 API 접근 권한 관리.
- 데이터 주입: 컨트롤러 전달 전 공통 데이터 추가.

### AOP
- 트랜잭션: "@Transactional" 관리.
- 성능 측정: 메서드 실행 시간 모니터링.
- 예외 복구: 비즈니스 로직 재시도 처리.
