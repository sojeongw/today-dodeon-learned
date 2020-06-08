# 프로젝트에 Spring Data JPA 적용하기

## 의존성 추가

{% tabs %}
{% tab title="build.gradle" %}
```groovy
buildscript {
    ext {
        springBootVersion = '2.1.7.RELEASE'
    }
    repositories {
        mavenCentral()
        jcenter()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:${springBootVersion}")
    }
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'
apply plugin: 'io.spring.dependency-management' 

group = 'com.jojoldu'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '1.8'

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
    jcenter()
}

dependencies {
    compile('org.projectlombok:lombok')
    compile('org.springframework.boot:spring-boot-starter-web')
    testCompile('org.springframework.boot:spring-boot-starter-test')

    // 스프링 부트용 Spring Data Jpa 추상화 라이브러리
    // 스프링 부트 버전에 맞춰 자동으로 JPA 관련 라이브러리들의 버전을 관리해준다.
    compile('org.springframework.boot:spring-boot-starter-data-jpa')

    // 인 메모리 관계형 데이터베이스
    // 별도의 설치 없이 프로젝트 의존성만으로 관리할 수 있다.
    // 메모리에서 실행되므로 애플리케이션을 재실행 할 때마다 초기화 된다.
    compile('com.h2database:h2')
}

test {
    useJUnitPlatform()
}
```
{% endtab %}
{% endtabs %}

먼저 Spring Data JPA와 H2 의존성을 추가한다.

## domain 패키지 추가

![](../../.gitbook/assets/freelac-jojoldu-spring-aws/03/스크린샷%202020-07-19%20오후%2010.15.23.png)

도메인을 담을 `domain` 패키지를 만든다. 

> 도메인이란 게시글, 댓글, 회원, 정산, 결제 등 소프트웨어에 대한 요구사항이나 문제 영역이라고 생각하면 된다.

{% tabs %}
{% tab title="Posts.java" %}
```java
@Getter
@NoArgsConstructor
@Entity // 주요 애너테이션을 클래스와 가깝게 해주면, 추후 delombok 할 때 편하다.
public class Posts {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(length = 500, nullable = false)
    private String title;

    @Column(columnDefinition = "TEXT", nullable = false)
    private String content;

    private String author;

    @Builder
    public Posts(String title, String content, String author) {
        this.title = title;
        this.content = content;
        this.author = author;
    }
}
```
{% endtab %}
{% endtabs %}

### @Getter

클래스 내 모든 필드의 getter 메서드를 자동으로 생성한다.

### @NoArgsConstructor

기본 생성자를 자동으로 추가한다. `public Posts() {}`와 같은 효과를 낸다.

### @Entity

테이블과 링크될 클래스임을 나타낸다. 기본값으로 클래스의 카멜 케이스 이름을 언더 스코어로 매칭한다.

### @Id

해당 테이블의 PK 필드를 나타낸다.

Entity의 PK는 웬만하면 `Long` 타입(MySQL 기준 bigint 타입)에 `auto_increment`가 좋다. PK를 주민등록번호처럼 비즈니스 상 유니크 키나, 여러 키를 조합한 복합키로 잡으면 아래의 경우가 발생한다.

1. FK를 맺을 때 다른 테이블에서 복합키 전부를 가지고 있거나, 중간 테이블을 하나 더 둬야 하는 상황이 발생한다.
2. 인덱스에 좋은 영향을 끼치치 못한다.
3. 유니크한 조건이 변경될 경우 PK 전체를 수정해야 한다.

따라서 주민등록번호, 복합키 등은 유니크 키로 별도 추가하는 것이 좋다.

### @GeneratedValue(strategy = GenerationType.IDENTITY)

PK의 생성 규칙을 나타낸다. 스프링 부트 2.0에서는 `IDENTITY` 옵션을 추가해야만 `auto_increment`가 된다. */

**Reference**

[스프링부트 2.0과 1.5의 차이](https://jojoldu.tistory.com/295)

### @Column

테이블의 칼럼을 나타낸다. 굳이 선언하지 않아도 해당 클래스의 필드는 모두 칼럼이 된다. 

그럼에도 사용하는 이유는, 기본값 외에 추가로 변경이 필요한 옵션이 있을 수 있기 때문이다. 문자열은 `varchar(255)`가 기본인데 사이즈를 늘리고 싶거나, 타입을 `TEXT`로 변경하고 싶을 때 등에 사용한다.

### @Builder

해당 클래스의 빌더 패턴 클래스를 생성한다. 생성자 상단에 선언하면 생성자에 포함된 필드만 빌더에 포함한다.

`Posts` 클래스에는 Setter가 없다. 무작정 생성하면 나중에 어디서 값이 변해야 하는지 코드상으로 명확히 구분할 수가 없기 때문이다. 그래서 **Entity 클래스에서는 절대 Setter 메서드를 만들지 않는다.**

{% tabs %}
{% tab title="잘못된 예" %}
```java
public class Order {
    public void setStatus(boolean status) {
        this.status = status;
    }
}

public void 주문서비스의_취소이벤트() {
    order.setStatus(false);
}
```
{% endtab %}

{% tab title="올바른 예" %}
```java
public class Order {
    public void cancelOrder() {
        this.status = false;
    }
}

public void 주문서비스의_취소이벤트() {
    order.cancelOrder();
}
```
{% endtab %}
{% endtabs %}

그럼 Setter가 없는 상황에서 어떻게 값을 채워 DB에 insert 해야 할까? 기본적으로는 생성자로 값을 채운 후 변경이 필요하다면 각 이벤트에 맞는 public 메서드를 호출한다.

여기서는 생성자 대신 빌더 클래스를 사용한다. 생성자와 빌더 모두 생성 시점에 값을 채우지만 생성자는 채워야할 필드가 무엇인지 명확히 지정할 수 없다.

{% tabs %}
{% tab title="생성자" %}
```java
// new Example(b, a)처럼 a와 b의 위치를 바꿔도 코드를 실행하기 전까지는 문제를 알 수 없다.
public Example(String a, String b) {
    this.a = a;
    this.b = b;
}
```
{% endtab %}

{% tab title="빌더" %}
```java
// 어떤 필드에 어떤 값을 채워야할지 명확하게 인지할 수 있다.
Example.builer()
    .a(a)
    .b(b)
    .build();
```
{% endtab %}
{% endtabs %}

## Repository 생성

{% tabs %}
{% tab title="PostsRepository" %}
```java
public interface PostsRepository extends JpaRepository<Posts, Long> {
}
```
{% endtab %}}
{% endtabs %}

`Repository`는 DB에 접근하는 인터페이스다. `JpaRepository<Entity 클래스, PK 타입>`을 상속하면 기본 CRUD 메서드가 자동으로 생성된다. `@Repository`를 추가할 필요도 없다.

Entity 클래스와 기본 Entity Repository는 함께 위치해야 한다. 둘은 아주 밀접한 관계이고 Entity는 기본 Repository 없이 제대로 역할을 수행할 수 없기 때문이다.

나중에 프로젝트가 커져 도메인별로 프로젝트를 분리해야 한다면 Entity 클래스와 기본 Repository는 함께 움직여야 하므로 도메인 페키지에서 함께 관리한다.

## 테스트 코드 작성

{% tabs %}
{% tab title="PostsRepositoryTest" %}
```java
import static org.assertj.core.api.Assertions.assertThat;

@RunWith(SpringRunner.class)
@SpringBootTest
public class PostsRepositoryTest {

    @Autowired
    PostsRepository postsRepository;

    @After
    public void cleanup() {
        postsRepository.deleteAll();
    }

    @Test
    public void 게시글저장_불러오기() {
        // given
        String title = "테스트 게시글";
        String content = "테스트 본문";

        postsRepository.save(Posts.builder().title(title).content(content).author("dodeon").build());

        // when
        List<Posts> postsList = postsRepository.findAll();

        // then
        Posts posts = postsList.get(0);
        assertThat(posts.getTitle()).isEqualTo(title);
        assertThat(posts.getContent()).isEqualTo(content);
    }
}
```
{% endtab %}}
{% endtabs %}

### @SpringBootTest

사용하면 별다른 설정없이 H2 데이터베이스를 자동으로 실행한다.

### After

Junit에서 단위 테스트가 끝날 때마다 수행되는 메서드를 지정한다.

보통은 배포 전 전체 테스트를 수행할 때 테스트 간에 데이터 침범을 막기 위해 사용한다. 여러 테스트가 동시에 수행되면 H2에 데이터가 그대로 남아 다음 테스트에 실패할 수 있다.

### save()

id 값이 있으면 update, 없다면 insert 쿼리가 실행된다.

## 쿼리 설정

### 쿼리 로그 확인

{% tabs %}
{% tab title="application.properties" %}
```properties
spring:
  jpa:
    show-sql: true
```
{% endtab %}}
{% endtabs %}

![](../../.gitbook/assets/freelac-jojoldu-spring-aws/03/스크린샷%202020-07-19%20오후%2010.58.56.png)

콘솔에서 쿼리를 직접 확인할 수 있다.

### MySQL 옵션 설정

그런데 위에서 create table 쿼리를 보면 `id bigint generated by default as identity`가 보인다. H2 쿼리 문법이 적용된 것이다. H2는 MySQL 문법을 적용할 수 있으므로 변경해보자.

{% tabs %}
{% tab title="application.properties" %}
```properties
spring:
  jpa:
    show-sql: true
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MySQL5InnoDBDialect
```
{% endtab %}}
{% endtabs %}

![](../../.gitbook/assets/freelac-jojoldu-spring-aws/03/스크린샷%202020-07-19%20오후%2011.03.27.png)

옵션이 잘 적용되었다.