# 순수 JPA 리포지토리와 Querydsl

{% tabs %} {% tab title="순수 JPA" %}

```java

@Repository
public class MemberJpaRepository {

    public List<Member> findAll() {
        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }

    public List<Member> findByUsername(String username) {
        return em.createQuery("select m from Member m where m.username   = :username", Member.class)
                .setParameter("username", username)
                .getResultList();
    }
}

@SpringBootTest
@Transactional
class MemberJpaRepositoryTest {

    @Test
    public void basicTest() {
        Member member = new Member("member1", 10);
        memberJpaRepository.save(member);

        Member findMember = memberJpaRepository.findById(member.getId()).get();
        assertThat(findMember).isEqualTo(member);

        List<Member> result1 = memberJpaRepository.findAll();
        assertThat(result1).containsExactly(member);

        List<Member> result2 = memberJpaRepository.findByUsername("member1");
        assertThat(result2).containsExactly(member);
    }
}
```

{% endtab %} {% tab title="Querydsl" %}

```java

@Repository
public class MemberJpaRepository {

    public List<Member> findAll_Querydsl() {
        return queryFactory
                .selectFrom(member).fetch();
    }

    public List<Member> findByUsername_Querydsl(String username) {
        return queryFactory
                .selectFrom(member)
                .where(member.username.eq(username))
                .fetch();
    }
}

@SpringBootTest
@Transactional
class MemberJpaRepositoryTest {

    @Test
    public void basicQuerydslTest() {
        Member member = new Member("member1", 10);
        memberJpaRepository.save(member);

        Member findMember = memberJpaRepository.findById(member.getId()).get();
        assertThat(findMember).isEqualTo(member);

        List<Member> result1 = memberJpaRepository.findAll_Querydsl();
        assertThat(result1).containsExactly(member);

        List<Member> result2 =
                memberJpaRepository.findByUsername_Querydsl("member1");
        assertThat(result2).containsExactly(member);
    }
}
```

{% endtab %} {% endtabs %}

- Querydsl를 쓰면 문자열로 짜는 JPQL과 달리 자바 코드로 짜기 때문에 오류를 빨리 발견할 수 있다.
- 쿼리 파라미터 바인딩도 신경쓰지 않아도 된다.

## 쿼리 팩토리 주입

{% tabs %} {% tab title="생성자 방식" %}

```java

@Repository
public class MemberJpaRepository {

    private final EntityManager em;
    private final JPAQueryFactory queryFactory;

    public MemberJpaRepository(EntityManager em) {
        this.em = em;
        this.queryFactory = new JPAQueryFactory(em);
    }
}
```

{% endtab %} {% tab title="생성자 방식" %}

```java

@SpringBootApplication
public class KimyounghanQuerydslApplication {

    public static void main(String[] args) {
        SpringApplication.run(KimyounghanQuerydslApplication.class, args);
    }

    @Bean
    JPAQueryFactory jpaQueryFactory(EntityManager em) {
        return new JPAQueryFactory(em);
    }
}

@Repository
public class MemberJpaRepository {

    private final EntityManager em;
    private final JPAQueryFactory queryFactory;

    // 이미 빈으로 등록됐기 때문에 바로 주입하면 된다.
    // 롬복 @RequiredConstructor로 대체할 수 있게 된다.
    public MemberJpaRepository(EntityManager em, JPAQueryFacotry queryFactory) {
        this.em = em;
        this.queryFactory = queryFactory;
    }
}
```

{% endtab %} {% endtabs %}

- queryFactory를 주입할 때는 생성자와 @Bean 방식 두 가지가 있다.

### 동시성

- 여기서 스프링이 주입해주는 엔티티 매니저는 프록시용 가짜 엔티티 매니저다.
    - 실제 동작 시점에 트랜잭션 단위로 실제 앤티티 매니저(영속성 컨텍스트)를 할당한다.
- 따라서 멀티 스레드 문제는 신경쓰지 않아도 된다.