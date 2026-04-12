# Servlet
자바진영에서의 HTTP 요청을 받아서 처리하고 응답을 만들어주는 서버 컴포넌트  
자바 표준(Jakarta EE / Java EE) 스펙이며, Tomcat과 같은 서블릿 컨테이너 위에서 동작한다.

## 핵심 객체
### HttpServletRequest
- HTTP 요청 메시지(Method, Header, Body, Query String 등)를 자바 객체로 추상화한 것.
- 생성: 서블릿 컨테이너(Tomcat 등)가 요청이 올 때마다 생성하여 서블릿에 전달.
- 역할: 개발자가 HTTP 프로토콜을 직접 파싱하지 않고도 요청 정보를 편리하게 조회 가능하게 한다.

### HttpServletResponse
- HTTP 응답 메시지(Status Code, Header, Body 등)를 생성하기 위한 객체.

## Life Cycle
1. init(): 서블릿 객체 생성 후 최초 1회 호출 (초기화 작업).
2. service(): 클라이언트 요청 시마다 호출(HTTP Method에 따라 doGet(), doPost() 등을 실행)
3. destroy(): 서블릿 종료 시 1회 호출(자원 해제)

## 멀티스레드의 지원
기본적으로 서블릿은 서버당 객체가 단 하나만 생성되어 공유됨  
서블릿은 멀티스레드를 지원. 요청마다 새로운 스레드가 생성되어 service() 메서드를 호출함.  
때문에 서블릿이 상태를 가지면 동시성 문제가 발생할 수 있음.  
따라서 인스턴스 변수를 사용하지 않고 지역변수를 사용해 스레드마다 독립을 시키는 설계가 필요.  
Spring의 @Controller, @Service등 서블릿과 동일한 구조이다.

## DispatcherServlet
- 내부적으로 HttpServlet을 상속받은 하나의 서블릿
- 모든 요청을 하나의 대표 서블릿이 받아 처리하는 Front Controller 패턴
