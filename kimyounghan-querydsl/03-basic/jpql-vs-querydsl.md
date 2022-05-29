# JPQL vs Querydsl

{% tabs %} {% tab title="QuerydslBasicTest.java" %}

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @PersistenceContext
    EntityManager em;

    @BeforeEach
    public void before() {
        Team teamA = new Team("teamA");
        Team teamB = new Team("teamB");

        em.persist(teamA);
        em.persist(teamB);

        Member member1 = new Member("member1", 10, teamA);
        Member member2 = new Member("member2", 20, teamA);
        Member member3 = new Member("member3", 30, teamB);
        Member member4 = new Member("member4", 40, teamB);

        em.persist(member1);
        em.persist(member2);
        em.persist(member3);
        em.persist(member4);
    }

    @Test
    void startJPQL() {
        String qlString = "select m from Member m " +
                "where m.username = :username";

        Member findMember = em.createQuery(qlString, Member.class)
                .setParameter("username", "member1")
                .getSingleResult();

        assertThat(findMember.getUsername()).isEqualTo("member1");
    }

    @Test
    public void startQuerydsl() {
        // 엔티티 매니터를 파라미터로 넘겨줘야 이걸 통해 데이터를 찾는다.
        JPAQueryFactory queryFactory = new JPAQueryFactory(em);

        // complieQuerydsl로 Q 클래스 생성이 확인됐다면 바로 사용할 수 있다.
        QMember m = new QMember("m");   // 구분하는 이름을 지정한다.

        Member findMember = queryFactory.select(m)
                .from(m)
                // JPQL과 달리 파라미터 바인딩을 지정해주지 않아도 알아서 바인딩한다.
                .where(m.username.eq("member1"))
                .fetchOne();

        assertThat(findMember.getUsername()).isEqualTo("member1");
    }
}

```

{% endtab %} {% endtabs %}

- EntityManager를 넘겨 JPAQueryFactory를 생성한다.
- JPQL
    - 문자 기반으로 작성한다.
        - 실행 시점에 오류를 발견하게 된다.
    - 파라미터 바인딩을 직접 한다.
- Querydsl
    - 코드 기반으로 작성한다.
        - 컴파일 시점에 오류를 발견할 수 있다.
    - 파라미터 바인딩을 자동으로 처리한다.

## JPAQueryFactory를 필드로 전환

{% tabs %} {% tab title="QuerydslBasicTest.java" %}

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @PersistenceContext
    EntityManager em;

    JPAQueryFactory queryFactory;

    @BeforeEach
    public void before() {
        // 다수의 스레드가 접근해도 문제 없게 설계되어 있다.
        queryFactory = new JPAQueryFactory(em);

        ...
    }

    @Test
    public void startQuerydsl2() {
        QMember m = new QMember("m");

        Member findMember = queryFactory
                .select(m)
                .from(m)
                .where(m.username.eq("member1"))
                .fetchOne();

        assertThat(findMember.getUsername()).isEqualTo("member1");
    }
}
```

{% endtab %} {% endtabs %}

- JPAQueryFactory를 공통으로 사용하도록 빼낼 수 있다.

### 동시성 문제

- JPAQueryFactory를 생성할 때 넘기는 EntityManager에 달려있다.
- 스프링은 여러 스레드에서 동시에 같은 EntityManager에 접근해도 트랜잭션마다 별도의 영속성 컨텍스트를 제공한다.
- 따라서 동시성 문제는 걱정하지 않아도 된다.