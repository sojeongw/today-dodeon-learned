# @EntityGraph

## N + 1 문제

```java

@SpringBootTest
@Transactional
@Rollback(value = false)
class MemberRepositoryTest {

    @Test
    void findMemberLazy() {
        Team teamA = new Team("teamA");
        Team teamB = new Team("teamB");

        teamRepository.save(teamA);
        teamRepository.save(teamB);

        memberRepository.save(new Member("member1", 10, teamA));
        memberRepository.save(new Member("member2", 20, teamB));

        em.flush();
        em.clear();

        List<Member> members = memberRepository.findAll();

        for (Member member : members) {
            member.getTeam().getName();
        }
    }
}
```

```sql
select member0_.member_id as member_i1_0_,
       member0_.age       as age2_0_,
       member0_.team_id   as team_id4_0_,
       member0_.username  as username3_0_
from member member0_;

select team0_.team_id as team_id1_1_0_, team0_.name as name2_1_0_
from team team0_
where team0_.team_id = 1;

select team0_.team_id as team_id1_1_0_, team0_.name as name2_1_0_
from team team0_
where team0_.team_id = 2;
```

- fetch 방식이 Lazy면 결과 개수에 따라 쿼리가 더 나간다.
- 네트워크를 그만큼 타는 것이기 때문에 성능 저하가 발생한다.

## fetch join

{% tabs %} {% tab title="MemberRepository.java" %}

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    // member를 조회할 때 연관된 team도 함께 가져온다.
    @Query("select m from Member m left join fetch m.team")
    List<Member> findMemberFetchJoin();

}
```

{% endtab %} {% tab title="MemberRepositoryTest.java" %}

```java

@SpringBootTest
@Transactional
@Rollback(value = false)
class MemberRepositoryTest {

    void findMemberLazy() {
        Team teamA = new Team("teamA");
        Team teamB = new Team("teamB");

        teamRepository.save(teamA);
        teamRepository.save(teamB);

        memberRepository.save(new Member("member1", 10, teamA));
        memberRepository.save(new Member("member2", 20, teamB));

        em.flush();
        em.clear();

        List<Member> members = memberRepository.findMemberFetchJoin();

        for (Member member : members) {
            member.getTeam().getName();
        }
    }
}
```

{% endtab %} {% endtabs %}

```sql
select member0_.member_id as member_i1_0_0_,
       team1_.team_id     as team_id1_1_1_,
       member0_.age       as age2_0_0_,
       member0_.team_id   as team_id4_0_0_,
       member0_.username  as username3_0_0_,
       team1_.name        as name2_1_1_
from member member0_
         left outer join team team1_ on member0_.team_id = team1_.team_id;
```

- fetch join을 명시해서 조회하면 프록시 없이 한 번에 값을 다 채워서 가져온다.
- 그런데 스프링 데이터 JPA는 자동으로 제공되는 메서드도 있어서 fetch join을 적용하기가 번거롭다.

## @EntityGraph

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    // 공통 메서드 오버라이드
    @Override
    @EntityGraph(attributePaths = {"team"})
    List<Member> findAll();

    // JPQL + 엔티티 그래프
    @EntityGraph(attributePaths = {"team"})
    @Query("select m from Member m")
    List<Member> findMemberEntityGraph();

    // 메서드 이름으로 쿼리 자동 생성할 때도 가능
    @EntityGraph(attributePaths = {"team"})
    List<Member> findByUsername(String username);

}
```

```sql
select member0_.member_id as member_i1_0_0_,
       team1_.team_id     as team_id1_1_1_,
       member0_.age       as age2_0_0_,
       member0_.team_id   as team_id4_0_0_,
       member0_.username  as username3_0_0_,
       team1_.name        as name2_1_1_
from member member0_
         left outer join team team1_ on member0_.team_id = team1_.team_id;
```

- fetch join을 편하게 적용할 수 있다.
- left outer join을 사용한다.
- 기본으로 제공되는 메서드에도 적용할 수 있다.

## @NamedEntityGraph

{% tabs %} {% tab title="Member.java" %}

```java

@Entity
@NamedEntityGraph(name = "Member.all", attributeNodes = @NamedAttributeNode("team"))
public class Member {
    
    ...
}
```

{% endtab %} {% tab title="MemberRepository.java" %}

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @EntityGraph("Member.all")
    @Query("select m from Member m")
    List<Member> findMemberEntityGraph();

}
```

{% endtab %} {% endtabs %}

```sql
select member0_.member_id as member_i1_0_0_,
       team1_.team_id     as team_id1_1_1_,
       member0_.age       as age2_0_0_,
       member0_.team_id   as team_id4_0_0_,
       member0_.username  as username3_0_0_,
       team1_.name        as name2_1_1_
from member member0_
         left outer join team team1_ on member0_.team_id = team1_.team_id;
```

- NamedQuery를 활용해서 fetch join 할 수도 있다.

## 활용

- 보통 직접 @Query에 JPQL로 fetch join을 넣어서 사용한다.
- 너무 간단해서 쓰기 귀찮으면 @EntityGraph를 쓴다.