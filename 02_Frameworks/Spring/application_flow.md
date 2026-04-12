# Spring 초기화 및 요청 처리 흐름
Spring 애플리케이션을 초기화 단계와 요청 처리 단계로 나누어 설명.  
초기화 단계에서는 애플리케이션 실행을 위한 환경을 준비, 요청 처리 단계에서는 클라이언트의 HTTP 요청을 처리하고 응답을 반환

## Spring 애플리케이션 초기화
### 1. Spring 애플리케이션 실행
- SpringApplication.run() 실행
- ApplicationContext 생성

### 2. BeanDefinition 로딩
- 설정 파일(XML, Java Config, Annotation)을 기반으로 Bean 정의 로딩
- BeanDefinition: Bean 생성 방법을 정의한 메타데이터

### 3. Bean 생성 및 의존성 주입 (DI)
- Bean 인스턴스 생성
- @Autowired 등을 통해 의존성 주입 수행

### 4. WebApplicationContext 구성
- 웹 환경에서는 WebApplicationContext 생성
- DispatcherServlet과 연결됨

### 5. DispatcherServlet 초기화
- HandlerMapping, HandlerAdapter, ViewResolver 등 초기화

### 6. 톰캣 실행
- 내장 톰캣 실행
- 포트 바인딩 (예: 8080)
- 클라이언트 요청 대기 상태 진입

---

## 사용자 요청 처리 흐름
### 1. 클라이언트의 HTTP 요청
- 브라우저, 모바일 앱, API 클라이언트가 서버로 HTTP 요청을 보낸다.
- 요청에는 다음 정보가 포함된다.
  - HTTP Method (GET, POST, PUT, DELETE 등)
  - URL
  - Header
  - Body
- 이 요청은 서버의 특정 포트(예: 8080)로 전달된다.

### 2. 톰캣(Tomcat, Servlet Container)의 요청 수신
- 톰캣이 서버 소켓을 통해 클라이언트의 TCP 연결을 수신한다.
- HTTP 프로토콜에 맞게 요청 데이터를 파싱한다.
- 요청을 처리할 워커 스레드를 Thread Pool에서 할당한다.
- 이후 요청/응답 정보를 다루기 위한 객체를 준비한다.
  - HttpServletRequest
  - HttpServletResponse
- 핵심은 톰캣이 네트워크 레벨의 요청을 받아서 자바 서블릿이 이해할 수 있는 형태로 바꿔준다는 것.

### 3. 서블릿 매핑 수행
- 톰캣은 요청 URL을 보고 어떤 서블릿이 이 요청을 처리할지 결정한다.
- 서블릿 매핑 규칙(exact, path, extension, default)을 사용한다.
- Spring MVC 환경에서는 일반적으로 DispatcherServlet이 "/" 에 매핑되어 대부분의 요청을 처리한다.
- 즉, 톰캣이 먼저 어떤 서블릿으로 보낼지 결정하고 Spring은 그 다음부터 동작함.

### 4. Filter Chain 실행
- 선택된 서블릿으로 보내기 전에 등록된 Filter들을 순서대로 실행한다.
- Filter는 서블릿 앞단에서 공통 처리를 담당한다.
- 인코딩 설정, CORS 처리, 로깅, 인증/인가, 보안 관련 처리 등이 여기서 일어난다.

### 5. DispatcherServlet 진입
- DispatcherServlet은 Spring MVC의 Front Controller로서 모든 요청의 공통 진입점 역할을 한다.
- 직접 비즈니스 로직을 수행하지 않고 적절한 핸들러(Controller)로 요청을 분배한다.

### 6. HandlerMapping으로 Controller 탐색
- DispatcherServlet은 요청 URL, HTTP Method 등을 기준으로 어떤 Controller가 이 요청을 처리해야 하는지 찾는다.
- HandlerMapping이 이 담당하며, 매핑된 메서드 정보를 반환한다.

### 7. Interceptor preHandle 실행
- Controller 호출 전 등록된 Interceptor가 있다면 preHandle()이 실행된다.
- 로그인 체크, 권한 확인, 요청 추적 등 Spring MVC 레벨의 공통 처리를 수행한다.
- 여기서 false를 반환하면 요청은 컨트롤러로 전달되지 않고 중단된다.

### 8. HandlerAdapter 및 Argument Resolver 동작
- DispatcherServlet은 찾은 Controller를 바로 호출하지 않고 HandlerAdapter를 통해 실행한다.
- 이 단계에서 Argument Resolver가 동작하여 다음 작업을 수행한다.
  - @RequestParam, @PathVariable 값 바인딩
  - @RequestBody JSON 데이터를 DTO로 변환 (Jackson 역직렬화)
  - HttpServletRequest, HttpSession 등 필요한 객체 주입

### 9. Controller 실행
- 실제 Controller 메서드가 호출되어 요청을 해석한다.
- 입력값에 대한 기본적인 검증을 수행하고 구체적인 작업은 Service 계층에 위임한다.

### 10. Service 계층 처리
- 비즈니스 로직을 수행하며, 보통 이 계층에서 트랜잭션(@Transactional) 경계가 설정된다.
- 도메인 규칙을 적용하고 필요한 데이터를 위해 Repository를 호출한다.

### 11. Repository / DB 접근
- Repository가 JPA, MyBatis 등을 통해 데이터베이스에 접근한다.
- SQL 실행 및 영속성 컨텍스트 작업을 통해 데이터 조회, 저장, 수정, 삭제를 수행한다.

### 12. 결과 반환 (Service -> Controller)
- DB 작업 결과가 Service를 거쳐 다시 Controller로 반환된다.
- Controller는 반환된 데이터를 바탕으로 최종 응답 형태(DTO, View 이름 등)를 결정한다.

### 13. Interceptor postHandle / afterCompletion 실행
- Controller 실행이 끝난 후 Interceptor 후처리가 실행된다.
- postHandle: 컨트롤러 로직 종료 후, 뷰 렌더링 전 시점에 실행된다.
- afterCompletion: 뷰 렌더링을 포함한 모든 처리가 완료된 후 실행된다.

### 14. 응답 변환 (MessageConverter / ViewResolver)
- Controller 반환값에 따라 최종 응답 메시지를 생성한다.
- @ResponseBody인 경우: HttpMessageConverter(Jackson 등)를 통해 객체를 JSON으로 직렬화한다.
- View 이름을 반환한 경우: ViewResolver가 해당 뷰를 찾아 템플릿 엔진으로 렌더링한다.

### 15. 톰캣의 HTTP Response 전송
- 최종적으로 만들어진 응답 데이터가 HttpServletResponse에 담긴다.
- 톰캣이 이를 HTTP 응답 메시지로 변환하여 클라이언트에게 전송한다.
- 요청 처리에 사용된 워커 스레드는 Thread Pool로 반환되어 재사용된다.

