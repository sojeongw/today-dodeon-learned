# Query By Example

{% tabs %} {% tab title="QueryByExampleTest.java" %}

```java

@SpringBootTest
@Transactional
public class QueryByExampleTest {

    @Autowired
    MemberRepository memberRepository;
    @Autowired
    EntityManager em;

    @Test
    public void basic() throws Exception {
        Team teamA = new Team("teamA");
        em.persist(teamA);
        
        em.persist(new Member("m1", 0, teamA));
        em.persist(new Member("m2", 0, teamA));
        em.flush();

        // Probe 생성
        Member member = new Member("m1");

        // 내부 조인으로 teamA 가능
        Team team = new Team("teamA");
        member.setTeam(team);

        // ExampleMatcher 생성
        // age 프로퍼티를 무시하도록 한다.
        ExampleMatcher matcher = ExampleMatcher.matching().withIgnorePaths("age");

        Example<Member> example = Example.of(member, matcher);
        List<Member> result = memberRepository.findAll(example);

        assertThat(result.size()).isEqualTo(1);
    }
}
```

{% endtab %} {% endtabs %}

- Probe
    - 필드에 데이터가 있는 실제 도메인 객체
- ExampleMatcher
    - 특정 필드를 일치시키는 상세한 정보 제공
    - 재사용 가능
- Example
    - Probe와 ExampleMatcher로 구성
    - 쿼리 생성에 사용

## 장점

- 동적 쿼리를 편리하게 처리한다.
- 도메인 객체를 그대로 사용한다.
- RDB에서 NoSQL로 변경해도 수정할 것 없게 추상화 되어 있다.
- 스프링 데이터 JPA의 JpaRepository 인터페이스에 이미 포함되어있다.

## 단점

- 내부 조인만 가능하고 외부 조인(left join)은 불가하다.
- 중첩 제약 조건이 불가하다.
    - firstname = ?0 or (firstname = ?1 and lastname = ?2)
- 매칭 조건이 매우 단순하다.

실무에서 사용하기엔 매칭 조건이 너무 단순하고 left 조인이 안되므로 실무에서는 QueryDSL을 사용한다.