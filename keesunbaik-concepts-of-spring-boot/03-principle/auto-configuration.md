# 자동 설정

## @SpringBootApplication

빈은 두 단계로 나눠서 읽힌다.

1. @ComponentScan
2. @EnableAutoConfiguration

### @SpringBootConfiguration

- 사실상의 @Configuration

### @ComponentScan

- 컴포넌트 애너테이션이 달린 클래스를 빈으로 등록한다.
- @Component, @Configuration, @Repository, @Service, @Controller, @RestController

### @EnableAutoConfiguration

- 컴포넌트 스캔으로 등록한 후 다시 한 번 빈을 읽어와 등록한다.
- 즉, 없어도 스프링 부트를 사용할 수 있다.
    - 단, 웹 애플리케이션 설정을 꺼줘야 가능
- 결국 이것도 @Configuration을 가지고 있다.
    1. spring.factories 라는 메타 파일을 읽어들인다.
    2. 메타 파일의 키 값을 보고 설정을 읽는다.
    3. 조건에 따라 빈이 생성되거나 생성되지 않는다.

## 예제

### 설정 만들기

우선 별도의 프로젝트를 새로 판다.

{% tabs %} {% tab title="pom.xml" %}

```xml

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-autoconfigure</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-autoconfigure-processor</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
<dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-dependencies</artifactId>
        <version>2.0.3.RELEASE</version>
        <type>pom</type>
        <scope>import</scope>
    </dependency>
</dependencies>
</dependencyManagement>
```

{% endtab %}{% tab title="Holoman.java" %}

```java
public class Holoman {

    String name;

    int howLong;

    @Override
    public String toString() {
        return "Holoman{" +
                "name='" + name + '\'' +
                ", howLong=" + howLong +
                '}';
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getHowLong() {
        return howLong;
    }

    public void setHowLong(int howLong) {
        this.howLong = howLong;
    }
}
```

{% endtab %} {% tab title="HolomanConfiguration.java" %}

```java

@Configuration
public class HolomanConfiguration {

    @Bean
    public Holoman holoman() {
        Holoman holoman = new Holoman();
        holoman.setHowLong(5);
        holoman.setName("Keesun");

        return holoman;
    }
}
```

{% endtab %} {% tab title="spring.factories" %}

```text
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  me.whiteship.HolomanConfiguration
```

{% endtab %} {% endtabs %}

1. 의존성을 추가한다.
2. @Configuration 파일을 작성한다.
3. `src/main/resource/META-INF`에 spring.factories를 생성한다.
4. 안에 자동 설정 파일을 추가한다.
5. `mvm install`을 실행한다.

### 설정 적용하기

{% tabs %} {% tab title="pom.xml" %}

```xml

<dependency>
    <groupId>me.whiteship</groupId>
    <artifactId>keesunbaik-spring-boot-starter</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/keesunbaik-concepts-of-spring-boot/03/스크린샷%202022-08-17%20오전%207.33.33.png)

기존 프로젝트로 돌아와 의존성을 적용하면 라이브러리가 나타난다.

{% tabs %} {% tab title="HolomanRunner.java" %}

```java

@Component
public class HolomanRunner implements ApplicationRunner {

    @Autowired
    Holoman holoman;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("holoman = " + holoman);
    }
}

```

{% endtab %} {% endtabs %}

```text
holoman = Holoman{name='Keesun', howLong=5}
```

Holoman을 어디서도 정의한 적이 없지만 사용할 수 있게 되었다.

### 설정 재정의 하기

```java

@SpringBootApplication
public class ConceptsOfSpringBootApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConceptsOfSpringBootApplication.class, args);
    }

    @Bean
    public Holoman holoman() {
        Holoman holoman = new Holoman();
        holoman.setName("whiteship");
        holoman.setHowLong(50);
        return holoman;
    }
}
```

하지만 빈을 다시 설정하려고 하면 기존 값 그대로 나온다

**스프링 부트 2.1부터는 오버라이딩 하지 못하고 에러가 뜬다.**

1. 컴포넌트 스캔이 먼저 동작한다
    - @Bean이 먼저 동작한다.
2. @AutoConfiguration으로 다시 등록한다.
    - @Bean 내용을 덮어 쓴다.

{% tabs %} {% tab title="HolomanConfiguration.java" %}

```java

@Configuration
public class HolomanConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public Holoman holoman() {
        Holoman holoman = new Holoman();
        holoman.setHowLong(5);
        holoman.setName("Keesun");

        return holoman;
    }
}

```

{% endtab %} {% endtabs %}

다시 설정 프로젝트 쪽으로 돌아가서 @ConditionalOnMissingBean을 달아준다.

- @ConditionalOnMissingBean
- 해당 타입의 빈이 없을 때만 이 빈을 등록하라는 뜻이 된다.

{% tabs %} {% tab title="pom.xml" %}

```xml

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

{% endtab %}{% tab title="application.properties" %}

```properties
holoman.name=whiteship
holoman.how-long=55
```

{% endtab %} {% tab title="HolomanProperties.java" %}

```java

@ConfigurationProperties("holoman")
public class HolomanProperties {

    private String name;

    private int howLong;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getHowLong() {
        return howLong;
    }

    public void setHowLong(int howLong) {
        this.howLong = howLong;
    }
}
```

{% endtab %}{% tab title="HolomanConfiguration.java" %}

```java

@Configuration
@EnableConfigurationProperties(HolomanProperties.class)
public class HolomanConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public Holoman holoman(HolomanProperties properties) {
        Holoman holoman = new Holoman();
        holoman.setHowLong(properties.getHowLong());
        holoman.setName(properties.getName());

        return holoman;
    }
}
```

{% endtab %} {% endtabs %}

- 설정 프로젝트에서 값을 설정할 때 빈에서 직접 하는 대신 properties에서 가져다 쓸 수도 있다.
- 본 프로젝트에서 properties에 값을 추가하면 그대로 반영된다.