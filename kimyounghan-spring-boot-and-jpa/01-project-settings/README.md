# 프로젝트 환경 설정

## 프로젝트 생성

[Spring Initializr](https://start.spring.io/) 에서 쉽게 프로젝트를 생성할 수 있다.

- Gradle
- Java
- Spring Boot 2.1.7
- groupId: jpabook
- artifactId: jpashop
- web, thymeleaf, jpa, h2, lombok, validation

## 라이브러리 살펴보기

### gradle 의존 관계 보기

```shell
./gradlew dependencies —configuration compileClasspath
```

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa/01/스크린샷%202021-03-10%20오후%2012.39.41.png)

### 스프링 부트 라이브러리

- spring-boot-starter-web
    - spring-boot-starter-tomcat
        - 톰캣 (웹서버)
    - spring-webmvc
        - 스프링 웹 MVC
- spring-boot-starter-thymeleaf
    - 타임리프 템플릿 엔진(View)
- spring-boot-starter-data-jpa
- spring-boot-starter-aop
- spring-boot-starter-jdbc
    - HikariCP 커넥션 풀 (부트 2.0 기본)
    - hibernate + JPA
        - 하이버네이트 + JPA
    - spring-data-jpa
        - 스프링 데이터 JPA
- spring-boot-starter(공통)
    - 스프링 부트 + 스프링 코어 + 로깅
    - spring-boot
        - spring-core
    - spring-boot-starter-logging
        - logback, slf4j

### 테스트 라이브러리

- spring-boot-starter-test
    - junit
        - 테스트 프레임워크
    - mockito
        - 목 라이브러리
    - assertj
        - 테스트 코드를 좀 더 편하게 작성하게 도와주는 라이브러리
    - spring-test
        - 스프링 통합 테스트 지원

- 핵심 라이브러리
    - 스프링 MVC
    - 스프링 ORM
    - JPA, 하이버네이트
    - 스프링 데이터 JPA

- 기타 라이브러리
    - H2 데이터베이스 클라이언트
    - 커넥션 풀
        - 부트 기본은 HikariCP
    - WEB(thymeleaf)
    - 로깅 SLF4J & LogBack
    - 테스트

스프링 데이터 JPA는 스프링과 JPA를 먼저 이해하고 사용해야 하는 응용기술이다.

## View 환경 설정

### thymeleaf 템플릿 엔진

[thymeleaf 공식 사이트](https://www.thymeleaf.org/)

[스프링 공식 튜토리얼](https://spring.io/guides/gs/serving-web-content/)

[스프링부트 메뉴얼](https://docs.spring.io/spring-boot/docs/2.1.6.RELEASE/reference/html/boot-features-developing-web-applications.html#boot-features-spring-mvc-template-engines)

`resources:template/` + `{ViewName}` + `.html` 경로를 통해 스프링 부트에 thymeleaf viewName이 매핑된다.

{% tabs %} {% tab title="HelloController.java" %}

```java

@Controller
public class HelloController {

  @GetMapping("hello")
  public String hello(Model model) {
    model.addAttribute("data", "hello!!");
    return "hello";
  }
}
```

{% endtab %} {% tab title="/templates/hello.html" %}

```thymeleafexpressions
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
<title>Hello</title>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
<p th:text="'안녕하세요. ' + ${data}" >안녕하세요. 손님</p>
</body>
</html>
```

{% endtab %} {% tab title="static/index.html" %}

```thymeleafexpressions
<!DOCTYPE HTML>
  <html xmlns:th="http://www.thymeleaf.org">
       
     <head>
        <title>Hello</title>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    </head>
    <body>
    Hello
    <a href="/hello">hello</a>
    </body>
</html>
```

{% endtab %} {% endtabs %}