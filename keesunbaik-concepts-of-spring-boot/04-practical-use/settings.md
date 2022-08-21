# 외부 설정

- properties
    - key-value 형태
- yaml
- 환경 변수
- @SpringBootTest 애노테이션의 properties 애트리뷰트

## 프로퍼티 우선 순위

1. 유저 홈 디렉토리에 있는 spring-boot-dev-tools.properties
2. 테스트에 있는 @TestPropertySource
3. @SpringBootTest 애노테이션의 properties 애트리뷰트
4. 커맨드 라인 아규먼트
5. SPRING_APPLICATION_JSON (환경 변수 또는 시스템 프로퍼티) 에 들어있는 프로퍼티
6. ServletConfig 파라미터
7. ServletContext 파라미터
8. java:comp/env JNDI 애트리뷰트
9. System.getProperties() 자바 시스템 프로퍼티
10. OS 환경 변수
11. RandomValuePropertySource
12. JAR 밖에 있는 특정 프로파일용 application properties
13. JAR 안에 있는 특정 프로파일용 application properties
14. JAR 밖에 있는 application properties
15. JAR 안에 있는 application properties
16. @PropertySource
17. 기본 프로퍼티 (SpringApplication.setDefaultProperties)

## @SpringBootTest 애노테이션의 properties 애트리뷰트

{% tabs %} {% tab title="application.properties" %}

```properties
keesun.name=whiteship
```

{% endtab %} {% tab title="SampleRunnerTest.java" %}

```java

@SpringBootTest
public class SampleRunnerTest {

    @Autowired
    Environment environment;

    @Test
    void test() {
        assertThat(environment.getProperty("keesun.name")).isEqualTo("whiteship");
    }
}
```

{% endtab %} {% endtabs %}

- 기존 proeperties에서 값을 가져온다.

{% endtab %} {% tab title="test/resources/" %}

```properties
keesun.name=dodeon
```

{% endtab %}{% tab title="SampleRunnerTest.java" %}

```java

@SpringBootTest
public class SampleRunnerTest {

    @Autowired
    Environment environment;

    @Test
    void test() {
        assertThat(environment.getProperty("keesun.name")).isEqualTo("dodeon");
    }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/keesunbaik-concepts-of-spring-boot/04/screenshot%202022-08-22%20오전%206.25.32.png)

- `/test/resources/`에 프로퍼티 설정을 따로 해주면 `/resource/` 값을 오버라이딩 한다.
- `/resource/`에만 추가한 값이 있다면 테스트를 실행할 때 오버라이딩 할 값이 없으므로 에러가 난다.

```java

@TestPropertySource(properties = "keesun.name=keesun2")
@SpringBootTest
public class SampleRunnerTest {

    @Autowired
    Environment environment;

    @Test
    void test() {
        assertThat(environment.getProperty("keesun.name")).isEqualTo("keesun2");
    }
}
```

- 테스트용 값을 따로 지정해주면 이 문제를 해결할 수 있다.

```java

@TestPropertySource(locations = "classpath:/test.properties")
@SpringBootTest
public class SampleRunnerTest {

    @Autowired
    Environment environment;

    @Test
    void test() {
        assertThat(environment.getProperty("keesun.name")).isEqualTo("keesun2");
    }
}
```

- test.properties를 따로 정의해서 사용해도 해결할 수 있다.

## 커맨드 라인 argument

```shell
java -jar target/springApplication-0.0.1.jar --keesun.name=whiteship
```

- 4순위이기 때문에 proeprties 값을 덮어 쓴다.

{% tabs %} {% tab title="application.properties" %}

```properties
keesun.name=whiteship
```

{% endtab %} {% tab title="SampleRunner.java" %}

```java

@Component
public class SampleRunner implements ApplicationRunner {

    @Value("${keesun.name}")
    private String name;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("name = " + name);
    }
}
```

{% endtab %} {% endtabs %}

## application.properties의 우선순위

1. file:./config/
2. file:./
3. classpath:/config/
4. classpath:/

## 기타 설정

- application.properties 랜덤값 설정
    - ${random.*}
- 플레이스 홀더 설정
    - name = keesun
    - fullName = ${name} baik

## @ConfigurationProperties

- 여러 프로퍼티를 묶어서 읽어올 수 있다.

{% tabs %} {% tab title="pom.xml" %}

```xml

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

{% endtab %} {% tab title="KeesunProperties.java" %}

```java

@Getter
@Setter
@Component
@ConfigurationProperties("keesun")
public class KeesunProperties {

    String name;

    int age;

    String fullName;
}
```

{% endtab %} {% tab title="SampleRunner.java" %}

```java

@Component
public class SampleRunner implements ApplicationRunner {

    @Autowired
    KeesunProperties keesunProperties;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("keesunProperties.name = " + keesunProperties.name);
        System.out.println("keesunProperties.age = " + keesunProperties.age);
        System.out.println("keesunProperties.fullName = " + keesunProperties.fullName);
    }
}
```

{% endtab %} {% endtabs %}

```text
keesunProperties.name = whiteship
keesunProperties.age = 892066220
keesunProperties.fullName = whiteship Baik
```

- properties에 등록된 값대로 잘 출력된다.
- @EnableConfigurationProperties
    - 메인 애플리케이션에 달고 Properties 클래스들을 등록해줘야 하지만 이미 자동으로 등록되므로 달아주지 않아도 된다.

## 융통성 있는 바인딩

- 프로퍼티 키 값의 형태는 다양할 수 있다.
    - context-path
    - context_path
    - contextPath

## @DurationUnit

{% tabs %} {% tab title="application.properties" %}

```properties
keesun.sessionTimeout=25
```

{% endtab %} {% tab title=".java" %}

```java

@Getter
@Setter
@Component
@ConfigurationProperties("keesun")
public class KeesunProperties {

    // 값이 안들어오면 기본값을 30초, 아니면 Seconds로 받겠다.
    @DurationUnit(ChronoUnit.SECONDS)
    private Duration sessionTimeout = Duration.ofSeconds(30);
}
```

{% endtab %} {% endtabs %}

```text
keesunProperties.getSessionTimeout = PT25S
```

- 시간 정보를 받고 싶을 때 사용한다.

{% tabs %} {% tab title="application.properties" %}

```properties
keesun.sessionTimeout=25s
```

{% endtab %} {% tab title=".java" %}

```java

@Getter
@Setter
@Component
@ConfigurationProperties("keesun")
public class KeesunProperties {

    private Duration sessionTimeout = Duration.ofSeconds(30);
}
```

{% endtab %} {% endtabs %}

```text
keesunProperties.getSessionTimeout = PT25S
```

- properties에 s를 붙여주면 애너테이션이 없어도 duration으로 알아서 인식한다.

## @Validated

- @NotNull 등 properties의 validation을 해준다.