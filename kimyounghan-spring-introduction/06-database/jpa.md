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

## 스프링 데이터 JPA

repository에 구현 클래스 없이 인터페이스만으로 개발을 완료할 수 있다. 기본 CRUD 기능도 모두 제공한다. 스프링 부트와 JPA 위에 스프링 데이터 JPA라는
프레임워크를 더하면 핵심 비즈니스 개발에 집중할 수 있다.

스프링 데이터 JPA는 JPA를 편리하게 사용하도록 도와주는 기술이다. 따라서 JPA를 먼저 학습한 후에 스프링 데이터 JPA를 학습해야 한다.

{% tabs %} {% tab title="SpringDataJpaMemberRepository.java" %}

```java
// 엔티티와 Id 타입을 맞춰 JpaRepository를 상속한 인터페이스를 만들어야 한다. 
// 그럼 스프링 데이터 JPA가 SpringDataJpaMemberRepository 빈을 자동으로 만들어준다.
// 우리가 만들었던 MemberRepository도 상속한다.
public interface SpringDataJpaMemberRepository extends JpaRepository<Member, Long>,
    MemberRepository {
  // 이렇게만 선언해주면 구현할 필요없이 쓸 수 있다.
  Optional<Member> findByName(String name);
}
```

{% endtab %} {% tab title="MemberService.java" %}

```java

@Configuration
public class SpringConfig {

  // 스프링 데이터 JPA가 만들어 놓은 구현체를
  private final MemberRepository memberRepository;

  // 주입받는다.
  public SpringConfig(MemberRepository memberRepository) {
    this.memberRepository = memberRepository;
  }

  // MemberService에 의존 관계를 세팅해준다.
  @Bean
  public MemberService memberService() {
    return new MemberService(memberRepository);
  }
}
```

{% endtab %} {% endtabs %}

## 스프링 데이터 JPA 기본 기능

![](../../.gitbook/assets/kimyounghan-spring-introduction/06/스크린샷%202021-03-09%20오전%208.57.22.png)

인터페이스를 통해 기본적인 CRUD와 페이징 기능을 쓸 수 있다. 하지만 `name`, `email`처럼 비즈니스 로직에 따라 다양해지는 내용은 인터페이스가 공통으로 제공할 수가 없다. 그래서 스프링 데이터 JPA는 `findByName()`, `findByEmail()`처럼 규칙에 따라 작성하면 JPQL로 `select m from MEmber m where m.name = ?`이라는 쿼리를 날려준다.

실무에서는 JPA와 스프링 데이터 JPA를 기본으로 사용하고, 복잡한 동적 쿼리는 QueryDsl이라는 라이브러리를 사용한다. QueryDsl을 사용하면 쿼리를 자바 코드로 안전하게 작성할 수 있다. 이 조합으로 해결하기 어려운 쿼리는 JPA가 제공하는 네이티브 쿼리를 사용하거나 JdbcTemplate을 사용하면 된다.