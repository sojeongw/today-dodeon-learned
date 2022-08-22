# 로깅

## 로깅 파사드

- 로깅 하는 부분을 추상화한 인터페이스
- 로거를 바꿔 낄 수 있는 장점이 있다.
- Commons Logging
    - 스프링 부트가 사용한다.
- SLF4j

둘 중 아무거나 써도 Log4J2를 사용하고 Log4J2가 Logback을 보낸다.

## 로거

- JUL
- Log4J2
- Logback
    - Log4J2의 구현체

최종적으로 우리는 Logback을 쓰고 있다.

## 기본 포맷

```text
--debug
```

- 일부 핵심 라이브러리만 디버깅 모드로 로깅한다.

```text
--trace
```

- 전부 다 디버깅 모드로 로깅한다.

### 컬러 출력

```properties
spring.output.ansi.enabled=always
```

### 파일 출력

```properties
logging.file={file}
logging.path={directory}
```

- logging.file 또는 logging.path로 로그를 파일로 저장할 수 있다.

### 로그 레벨 조정

```text
logging.level.{package}={log level}
```

- 패키지마다 로그 레벨을 조정할 수 있다.

## 커스텀 로그 설정

### logback

- logback-spring.xml
- extension을 지원해서 logback을 추천한다.
    - logback.xml은 너무 일찍 로딩이 되기 때문에 logback-spring.xml을 써야 적용된다.

#### extension

```xml

<springProfile name="{profile}"/>
```

- 프로파일 설정

```xml

<springProperty></springProperty>
```

- Environment 프로퍼티 설정

### log4j2

- log4j2-spring.xml

[로거를 Log4j2로 변경하기](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html#howto-configure-log4j-for-logging)

### JUL

- logging.properties
- 비추
