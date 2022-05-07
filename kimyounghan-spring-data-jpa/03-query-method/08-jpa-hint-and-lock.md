# JPA Hint & Lock

## JPA Hint

- SQL 힌트가 아니라 JPQ 구현체에게 제공하는 힌트

{% tabs %} {% tab title="MemberRepository.java" %}

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @QueryHints(value = @QueryHint(name = "org.hibernate.readOnly", value = "true"))
    Member findReadOnlyByUsername(String username);

}
```

{% endtab %} {% tab title="MemberRepositoryTest.java" %}

```java

@SpringBootTest
@Transactional
@Rollback(value = false)
class MemberRepositoryTest {

    @Test
    public void queryHint() throws Exception {
        memberRepository.save(new Member("member1", 10));

        // DB에 동기화
        em.flush();
        // 영속성 컨텍스트 초기화
        em.clear();

        // 영속성 컨텍스트를 초기화 했으니 무조건 DB에 쿼리를 날린다.
        Member member = memberRepository.findReadOnlyByUsername("member1");
        member.setUsername("member2");

        // 더티 체킹이 반영되지 않는다.
        em.flush();
    }
}
```

{% endtab %} {% endtabs %}

```sql
insert into member (age, team_id, username, member_id)
values (10, NULL, 'member1', 1);

select member0_.member_id as member_i1_0_,
       member0_.age       as age2_0_,
       member0_.team_id   as team_id4_0_,
       member0_.username  as username3_0_
from member member0_
where member0_.username = 'member1';
```

- 더티 체킹은 원본과 수정본을 둘 다 메모리에 들고 있어야 하고 체크하는 과정도 필요하기 때문에 비용이 든다.
    - 단지 조회만 하고 끝내고 싶어도 find()를 하는 순간 스냅샷을 떠놓게 된다.
- readOnly 힌트를 넘기면 select만 나가고 update는 나가지 않는다.
- 성능은 복잡한 쿼리 때문이지 readOnly를 적용하지 않아서는 거의 없다.
    - 성능 테스트를 해보고 정말 이점이 있는 곳에서 사용한다.

## Lock

{% tabs %} {% tab title="MemberRepository.java" %}

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @Lock(LockModeType.PESSIMISTIC_WRITE)
    List<Member> findByUsername(String name);

}
```

{% endtab %} {% tab title="MemberRepositoryTest.java" %}

```java

@SpringBootTest
@Transactional
@Rollback(value = false)
class MemberRepositoryTest {

    @Test
    public void lock() {
        memberRepository.save(new Member("member1", 10));

        em.flush();
        em.clear();

        List<Member> members = memberRepository.findByUsername("member1");
    }
}
```

{% endtab %} {% endtabs %}

```sql
select member0_.member_id as member_i1_0_,
       member0_.age       as age2_0_,
       member0_.team_id   as team_id4_0_,
       member0_.username  as username3_0_
from member member0_
where member0_.username = 'member1' for update;
```

- 락을 적용할 수 있다.
- for update
    - 데이터를 수정하려고 select 하는 중이니 다른 사람은 데이터에 손대지 말라는 뜻
- 실시간 트래픽이 많은 곳에 락을 걸면 위험하다.