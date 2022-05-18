# Specifications

- DDD 책을 보면 명세라는 개념을 소개한다.
    - 언어 상관없이 and, or 조건을 조합해 쿼리할 수 있도록 하는 기능
- 스프링 데이터 JPA는 JPA Criteria를 활용해 명세를 지원한다.

## Predicate

- org.springframework.data.jpa.domain.Specification
- 참 또는 거짓으로 평가한다.
- and, or 같은 연산자로 조합해서 다양한 검색 조건을 쉽게 생성한다.

```java
public interface MemberRepository extends JpaSpecificationExecutor<Member> {
}
```

```java
public interface JpaSpecificationExecutor<T> {

    Optional<T> findOne(@Nullable Specification<T> spec);

    List<T> findAll(Specification<T> spec);

    Page<T> findAll(Specification<T> spec, Pageable pageable);

    List<T> findAll(Specification<T> spec, Sort sort);

    long count(Specification<T> spec);
}

```

- JpaSpecificationExecutor를 상속하면 사용할 수 있다.

{% tabs %} {% tab title="MemberRepositoryTest.java" %}

```java
class MemberRepositoryTest {

    @Test
    public void specBasic() throws Exception {
        Team teamA = new Team("teamA");
        em.persist(teamA);
        Member m1 = new Member("m1", 0, teamA);

        Member m2 = new Member("m2", 0, teamA);
        em.persist(m1);
        em.persist(m2);
        em.flush();
        em.clear();

        Specification<Member> spec = MemberSpec.username("m1").and(MemberSpec.teamName("teamA"));
        List<Member> result = memberRepository.findAll(spec);

        Assertions.assertThat(result.size()).isEqualTo(1);
    }
}
```

{% endtab %} {% tab title="MemberSpec.java" %}

```java
public class MemberSpec {

    public static Specification<Member> teamName(final String teamName) {

        // predicate를 만든다.
        return (Specification<Member>) (root, query, builder) -> {
            if (StringUtils.isEmpty(teamName)) {
                return null;
            }

            // 회원과 조인한다.
            Join<Member, Team> t = root.join("team", JoinType.INNER);
            return builder.equal(t.get("name"), teamName);
        };
    }

    public static Specification<Member> username(final String username) {
        return (Specification<Member>) (root, query, builder) -> builder.equal(root.get("username"), username);
    }
}

```

{% endtab %} {% endtabs %}

- 실무에서는 QueryDSL을 더 많이 쓰기 때문에 참고만 한다.