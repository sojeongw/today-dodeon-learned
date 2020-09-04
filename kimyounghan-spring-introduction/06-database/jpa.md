# JPA

- 기존의 반복 코드는 물론이고 기본적인 SQL도 JPA가 직접 만들어서 실행해준다.
- SQL과 데이터 중심의 설계에서 객체 중심의 설계로 패러다임을 전환할 수 있다.
- 개발 생산성을 크게 높일 수 있다.

{% tabs %} {% tab title="build.gradle" %}

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    // implementation 'org.springframework.boot:spring-boot-starter-jdbc' 
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'  // jpa에 jdbc도 포함되어 있다.
    runtimeOnly 'com.h2database:h2'
    testImplementation('org.springframework.boot:spring-boot-starter-test') {
        exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
    }
}
```

{% endtab %} {% tab title="application.properties" %}

```properties
spring.datasource.url=jdbc:h2:tcp://localhost/~/test
spring.datasource.driver-class-name=org.h2.Driver
# 스프링부트 2.4부터는 꼭 추가해줘야 오류가 발생하지 않는다.
spring.datasource.username=sa
# jpa 설정 추가
# JPA가 생성하는 SQL을 호출한다.
spring.jpa.show-sql=true
# JPA는 테이블을 자동으로 생성하는 기능을 제공한다.
# none으로 설정하면 해당 기능을 끈다.
# create를 사용하면 엔티티 정보를 바탕으로 테이블을 직접 생성한다.
spring.jpa.hibernate.ddl-auto=none
```

{% endtab %} {% endtabs %}

JPA를 사용하려면 일단 엔티티를 매핑해야 한다. jpa는 인터페이스고 구현체로 hibernate가 있다. 따라서 hibernate 라이브러리가 필요하다.

JPA는 ORM(Object Relational Mapping) 즉, 객체와 데이터베이스 테이블을 매핑해주는 것이다.

{% tabs %} {% tab title="Member.java" %}

```java
// 테이블을 매핑해주는 애너테이션 = JPA가 관리하는 엔티티라는 표시 
@Entity
public class Member {

  // PK 매핑
  @Id
  // 아이디를 디비에서 자동으로 만들어주는 전략을 설정한다.
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;
  private String name;

  public Long getId() {
    return id;
  }

  public void setId(Long id) {
    this.id = id;
  }

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }
}

```

{% endtab %} {% tab title="JpaMemberRepository.java" %}

```java
public class JpaMemberRepository implements MemberRepository {

  // JPA는 Entity Manager로 동작한다. 
  // jpa 라이브러리를 추가해놓기만 하면 스프링 부트가 자동으로 엔티티 매니저를 생성해준다.
  // 우리는 만들어진 것을 주입만 받으면 된다.
  // 이전에 datasource로 했던 것(DB 연결 등)을 엔티티 매니저가 다 관리한다.
  private final EntityManager em;

  public JpaMemberRepository(EntityManager em) {
    this.em = em;
  }

  public Member save(Member member) {
    em.persist(member);
    return member;
  }

  public Optional<Member> findById(Long id) {
    Member member = em.find(Member.class, id);
    return Optional.ofNullable(member);
  }

  public Optional<Member> findByName(String name) {
    // name으로 찾아야 할 때는 JPQL이라는 객체지향 쿼리를 사용해야 한다.
    // Member라는 테이블이 아닌 엔티티를 대상으로 쿼리를 날린다.
    // m은 객체 자체를 select 하는 것이다.
    List<Member> result = em
        .createQuery("select m from Member m where m.name = :name ", Member.class)
        .setParameter("name", name)
        .getResultList();
    return result.stream().findAny();
  }

  public List<Member> findAll() {
    return em.createQuery("select m from Member m", Member.class)
        .getResultList();
  }
}
```

{% endtab %} {% tab title="MemberService.java" %}

```java
// 데이터를 저장하고 변경할 때는 항상 서비스 쪽에 @Transactional 애너테이션이 있어야 한다.
// 스프링은 해당 클래스의 메서드를 실행할 때 트랜잭션을 시작한다.
// 메서드가 정상 종료되면 트랜잭션을 커밋하고 런타임 예외가 발생하면 롤백한다.
// JPA를 통한 모든 데이터 변경은 트랜잭션 안에서 실행해야 한다.
@Transactional
public class MemberService {

}
```

{% endtab %} {% tab title="MemberService.java" %}

```java

@Configuration
public class SpringConfig {

  private final EntityManager em;

  public SpringConfig(EntityManager em) {
    this.em = em;
  }

  @Bean
  public MemberService memberService() {
    return new MemberService(memberRepository());
  }

  @Bean
  public MemberRepository memberRepository() {
//    return new MemoryMemberRepository();
//    return new JdbcMemberRepository(dataSource);
//    return new JdbcTemplateMemberRepository(dataSource);
    return new JpaMemberRepository(em);
  }
}
```

{% endtab %}{% endtabs %}