# vs Spring

## 1. 설정 파일의 차이

### Spring MVC

- web.xml을 통해 DispatcherServlet(이하 D.S.)을 설정하고 ApplicationContext(이하 A.C.)를 초기화.
- root-context.xml, servlet-context.xml을 직접 작성

### Spring Boot

@SpringBootApplication을 사용하면 자동으로 A.C.를 생성하며, 내장 톰켓을 포함하므로 별도 설정이 필요 없다. (SpringApplication.run 실행시 내장톰켓 실행 및 A.C. 생성)

#### @SpringBootApplicatoin

`@Configuration`: Spring의 XML 설정을 Java로 대체, 직접 Bean을 등록할 수 있음
`@EnableAutoConfiguraion`: Spring Boot가 필요할만한 Bean을 자동등록
`@ComponentScan`: 패키지 내의 @Component, @Service, @Controller, @Repository 자동 검색 및 등록

## 2. 의존성 관리 방식

### Spring MVC

- 의존성을 직접 설정해야 하며, 여러 개의 라이브러리를 개별적으로 추가해야 한다.

```
// xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.3.10</version>
</dependency>
```

### Spring Boot

- spring-boot-starter-* 라이브러리를 통해 필요한 기능을 자동으로 포함

```
// xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

## 3. 내장 서버 제공

### Spring MVC

- 일반적으로 Tomcat, Jetty등의 WAS(Web Application Server에 배포해야 한다.)

### Spring Boot

- spring-boot-starter-web을 사용하면 내장 톰캣을 자동 실행

## 기타등등

### Configuration

- SpringMVC는 XML기반 설정이 일반적이며 @Configuration을 사용해 Java기반 설정도 가능
- Spring boot는 기본적으로 자동 설정을 사용하며 필요한 설정은 properties, yml을 이용해 설정 변경 가능.

# 요약

- Spring Boot는 설정을 Map 처럼 관리하고 자동으로 Bean을 등록한다.
- 개발자는 최소한의 설정만 하면 되고, 나머지는 AutoConfiguration이 해결한다.

---

# 3.x 버전 주요 변경점

# Java 버전

Spring Boot 2.x 는 Java 8, 11 을 권장 Spring boot 3 부터 Java17 이상을 요구

# Jakarta EE 9 마이그레이션 ( javax.* -> jakarta.* )

엔터프라이즈 애플리케이션의 표준인 javax의 이름이 jakarta로 변경

# GraalVM 네이티브 이미지 지원

GraalVM은 고성능 다국어를 지원하는 차세대 JVM

기존 JVM의 흐름

- Java 코드 작성
- javac(Java Compiler)이 코드를 바이트코드(.class)로 변환 (Maven, Gradle과 같은 빌드 도구 실행하면 내부적으로 javac 멸령 실행)
- 빌드 tool이 컴파일된 바이트코드와 필요한 리소스를 하나로 묶어 .jar 또는 .war 로 만든다.
- Jar 파일을 실행하면 JVM이 구동되고 JVM은 바이트코드를 읽어 OS가 이해할 수 있는 기계어 코드로 변환하고 실행.

  GraalVM은 빌드 타임에 `./gradlew nativeCompile` 명령으로 네이티브 실행 파일을 생성(AOT 컴파일) 후 JVM없이 바로 실행이 가능.

현대의 MSA, 쿠버네티스 환경의 애플리케이션은 GraalVM이 유리하다.

# Observability 개선

## Micrometer의 @SpanTag를 지원

분산트레이싱을 위해서 사용. 예를 들어 요청으로 들어온 userID 또는 특정 주문번호를 @SpanTag로 지정하여 트레이싱 시스템에서 요청의 흐름을 빠르게 찾을 수 있다. -> 디버깅에 용이

## process InfoContributor 추가

Spring Boot Actuator가 제공하는 애플리케이션 정보로 앤드포인트에 프로세스 관련 상세 정보를 추가하는 기능 현재 실행중인 JVM 프로세스 자체의 정보를 포함시킬 수 있다.

제공되는 정보

- PID
- startTime
- path

운영환경에서 실행중인 특정 인스턴스를 식별하거나, 재시작 시간 등 운영에 필요한 정보를 얻을 수 있다.

# Spring Security 와 JWT 통합

JWT토큰을 Spring Security의 Authentication 객체로 변환하는 JwtAuthenticationConverter Bean 등록을 yml 파일에서 속성 설정으로 가능해짐.

이전 방식

```java

@Bean
public JwtAuthenticationConverter jwtAuthenticationConverter() {
    JwtGrantedAuthoritiesConverter grantedAuthoritiesConverter = new JwtGrantedAuthoritiesConverter();
    grantedAuthoritiesConverter.setAuthorityPrefix("ROLE_");
    grantedAuthoritiesConverter.setAuthoritiesClaimName("roles");
    JwtAuthenticationConverter jwtAuthenticationConverter = new JwtAuthenticationConverter();
    jwtAuthenticationConverter.setJwtGrantedAuthoritiesConverter(grantedAuthoritiesConverter);
    return jwtAuthenticationConverter;
}
```

3.3.0 이후

```yml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: https://your-auth-server.com
          authority-prefix: "ROLE_" # 권한 prefix 설정
          authorities-claim-name: "roles" # JWT의 권한 claim 이름
          principal-claim-name: "sub" # principal로 사용할 claim
```

## Testcontainers & Docker Compose 통합 확장

테스트 및 로커 개발 환경 개선

- testcontainers: 테스트 코드 실행 시 필요한 외부 서비스를 Docker 컨테이너로 자동으로 띄우고, Spring 애플리케이션에 연결 정보(URL, 포트 등)를 주입하는 기능 강화
- docker-compose.yaml 파일을 프로젝트 루트에 두면, Spring Boot가 이를 인식하고 로컬 개발 시 자동으로 서비스를 시작/중지하는 기능이 도입