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

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-development/01/스크린샷%202021-03-10%20오후%2012.39.41.png)

```groovy
plugins {
    id 'org.springframework.boot' version '2.4.3'
    id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    id 'java'
}

group = 'jpabook'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

dependencies {
    // 버전이 없는 라이브러리는 스프링 부트 버전과 궁합이 맞는 것을 자동으로 가져온다.
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
    implementation 'org.springframework.boot:spring-boot-starter-validation'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-devtools'
    compileOnly 'org.projectlombok:lombok'
    runtimeOnly 'com.h2database:h2'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation("org.junit.vintage:junit-vintage-engine") {
        exclude group: "org.hamcrest", module: "hamcrest-core"
    }
}

test {
    useJUnitPlatform()
}

```

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

{% endtab %} {% tab title="/static/index.html" %}

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

`resources:template/` + `{ViewName}` + `.html` 경로를 통해 스프링 부트에 thymeleaf viewName이 매핑된다. 기본 정적 화면은 `/static/index.html`에
정의한다.

`spring-boot-devtools` 라이브러리를 추가한 뒤 html 파일을 컴파일만 하면 서버 재시작 없이 변경 사항이 반영된다. `build-recompile` 메뉴에서 컴파일 할 수 있다.

## H2 데이터베이스 설치

개발이나 테스트 용도로 사용할 수 있는 가볍고 편리한 데이터베이스다. 웹 화면을 제공한다.

{% tabs %} {% tab title="/resources/application.yml" %}

```yaml
spring:
  datasource:
    url: jdbc:h2:tcp://localhost/~/jpashop
    username: sa
    password:
    driver-class-name: org.h2.Driver
  jpa:
    hibernate:
      ddl-auto: create  # 애플리케이션 실행 시점에 테이블을 drop하고 다시 생성한다.
    properties:
      hibernate:
        #      show_sql: true   # System.out에 하이버네이트 실행 SQL을 남긴다.
        format_sql: true
logging.level:
  org.hibernate.SQL: debug  # logger를 통해 하이버네이트 실행 SQL을 남긴다.
```

{% endtab %} {% endtabs %}

운영 환경의 모든 로그는 가급적 로거를 통해 남겨야 한다. `show_sql`은 `System.out`에서 찍기 때문에 가급적 안 쓰는 게 좋다.

`application.yml`같은 `yml` 파일은 띄어쓰기 2칸으로 계층을 만든다. 따라서 반드시 맞춰줘야 한다.

## JPA와 DB 설정, 동작 확인

{% tabs %}{% tab title="Member.java" %}

```java

@Entity
@Getter
@Setter
public class Member {

    @Id
    @GeneratedValue
    private Long id;
    private String username;
}
```

{% endtab %} {% tab title="MemberRepository.java" %}

```java

@Repository
public class MemberRepository {

    // 스프링 컨테이너가 EntityManager를 주입할 수 있게 해주는 애너테이션
    @PersistenceContext
    private EntityManager em;

    public Long save(Member member) {
        em.persist(member);

        // 커맨드와 쿼리를 분리하자.
        // 저장은 사이드 이펙트를 일으키는 커맨드 성이기 때문에 리턴을 안 하거나 아이디 정도만 반환한다.
        return member.getId();
    }

    public Member find(Long id) {
        return em.find(Member.class, id);
    }
}

```

{% endtab %} {% tab title="MemberRepositoryTest.java" %}

```java
// 스프링과 관련된 테스트를 할 것이라고 알려준다.
@RunWith(SpringRunner.class)
// 스프링 부트 프로젝트이므로 함께 넣어줘야 한다.
@SpringBootTest   
public class MemberRepositoryTest {

    @Autowired
    MemberRepository memberRepository;

    @Test
    // 추가하지 않으면 
    // No EntityManager with actual transaction available for current thread
    // - cannot reliably process 'persist' call 에러가 발생한다.
    // 엔티티의 데이터 변경은 모두 트랜잭션 안에서 이루어져야 하는데 트랜잭션이 없어서 나는 에러다.
    // @Transactional은 테스트에 있으면 테스트가 끝난 뒤 바로 롤백한다.
    @Transactional
    // 테스트 후 데이터를 날리고 싶지 않으면 이 옵션을 추가한다.
    @Rollback(value = false)
    public void save() {
        // given
        Member member = new Member();
        member.setUsername("memberA");

        // when
        Long savedId = memberRepository.save(member);
        Member findMember = memberRepository.find(savedId);

        // then
        Assertions.assertThat(findMember.getId()).isEqualTo(member.getId());
        Assertions.assertThat(findMember.getUsername()).isEqualTo(member.getUsername());

        // 같은 트랜잭션에 묶여있기 때문에 같은 영속성 컨텍스트에 존재한다.
        // 같은 영속성 컨텍스트에서는 아이디 값이 같으면 같은 Entity로 식별한다.
        // 이미 같은 영속성 컨텍스트에서 관리되고 있는 Entity가 있기 때문에 1차 캐시에서 가져온다.
        // 따라서 둘을 비교하면 true가 나온다.
        Assertions.assertThat(findMember).isEqualTo(member);
    }

    @Test
    public void find() {
    }
}
```

{% endtab %} {% endtabs %}

### 쿼리 파라미터 로그 남기기

기본 SQL 로그는 파라미터가 안 나와서 답답할 때가 많다.

{% tabs %} {% tab title="/resources/application.yml" %}

```yaml
spring:
  datasource:
    url: jdbc:h2:tcp://localhost/~/jpashop
    username: sa
    password:
    driver-class-name: org.h2.Driver
  jpa:
    hibernate:
      ddl-auto: create
    properties:
      hibernate:
        format_sql: true
logging.level:
  org.hibernate.SQL: debug
  # 파라미터를 로기로 남기는 옵션을 추가한다.
  org.hibernate.type: trace

```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-development/01/스크린샷%202021-03-11%20오전%2010.38.20.png)

아래에 각 파라미터의 값을 출력해준다.

```groovy
implementation 'com.github.gavlyukovskiy:p6spy-spring-boot-starter:1.5.6'
```

외부 라이브러리를 사용하는 방법도 있다.

[spring-boot-data-source-decorator](https://github.com/gavlyukovskiy/spring-boot-data-source-decorator)

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-development/01/스크린샷%202021-03-11%20오전%2010.44.07.png)

파라미터 값이 출력되는 것을 볼 수 있다.

참고로 운영 시스템에 외부 라이브러리를 쓸 때는 꼭 성능 테스트를 해봐야 한다.
