# 조인

## 기본 조인

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    /**
     * 팀A에 소속된 모든 회원
     */
    @Test
    public void join() throws Exception {
        QMember member = QMember.member;
        QTeam team = QTeam.team;

        List<Member> result = queryFactory
                .selectFrom(member)
                // 멤버와 팀을 조인한다.
                .join(member.team, team)
                .where(team.name.eq("teamA"))
                .fetch();

        assertThat(result).extracting("username")
    }
}
```

```text
join(조인 대상, 별칭으로 사용할 Q타입)
```

```sql
select member1
from Member member1
         inner join
     member1.team as team
where team.name = ?1
```

- join(), innerJoin()
    - inner join
- leftJoin()
    - left outer join
- rightJoin()
    - right outer join
- JPQL의 on과 성능 최적화를 위한 fetch join도 제공한다.

## 세타 조인

- 연관 관계가 없는 필드로 조인한다.

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    /**
     * 세타 조인(연관 관계가 없는 필드로 조인)
     * 회원의 이름이 팀 이름과 같은 회원 조회
     */
    @Test
    public void theta_join() throws Exception {
        em.persist(new Member("teamA"));
        em.persist(new Member("teamB"));

        List<Member> result = queryFactory
                .select(member)
                // 일반 조인은 멤버와 연관이 있는 팀을 지정했지만
                // 세타 조인은 그냥 필요한 엔티티를 나열한다.
                .from(member, team)
                .where(member.username.eq(team.name))
                .fetch();

        assertThat(result)
                .extracting("username")
                .containsExactly("teamA", "teamB");
    }
}
```

- from에 여러 엔티티를 선택해서 세타 조인 한다.
- outer join은 불가하다.
    - on을 사용하면 outer join이 가능해진다.

## on절

### 조인 대상 필터링

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Test
    public void join_on_filtering() throws Exception {
        List<Tuple> result = queryFactory
                .select(member, team)
                .from(member)
                .leftJoin(member.team, team)
                .on(team.name.eq("teamA"))
                .fetch();

        for (Tuple tuple : result) {
            System.out.println("tuple = " + tuple);
        }
    }
}
```

```sql
SELECT m.*, t.*
FROM Member m
         LEFT JOIN Team t ON m.TEAM_ID = t.id and
                             t.name = 'teamA'
```

- 회원과 팀을 조인하면서 팀 이름이 teamA인 것만 조인한다.
- 회원은 모두 조회한다.
- inner join을 사용하면 where에서 필터링하는 것과 동일하다.
    - inner join이라면 익숙한 where절로 해결한다.
    - outer join이 필요한 경우에만 on절을 사용한다.

```text
tuple = [Member(id=3, username=member1, age=10), Team(id=1, name=teamA)]
tuple = [Member(id=4, username=member2, age=20), Team(id=1, name=teamA)]
tuple = [Member(id=5, username=member3, age=30), null]
tuple = [Member(id=6, username=member4, age=40), null]
```

- left join이므로 teamB는 null인 상태로 조회한다.

### 연관 관계 없는 엔티티 외부 조인

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Test
    public void join_on_no_relation() throws Exception {
        em.persist(new Member("teamA"));
        em.persist(new Member("teamB"));

        List<Tuple> result = queryFactory
                .select(member, team)
                .from(member)
                // 일반 조인과 다르게 엔티티 하나만 들어간다.
                .leftJoin(team)
                .on(member.username.eq(team.name))
                .fetch();

        for (Tuple tuple : result) {
            System.out.println("t=" + tuple);
        }
    }
}
```

```sql
SELECT m.*, t.*
FROM Member m
         LEFT JOIN Team t ON m.username = t.name
```

- 회원의 이름과 팀의 이름이 같은 데이터를 outer join 한다.
- 하이버네이트 5.1부터 관계 없는 필드로도 on으로 외부 조인할 수 있다.
- 일반 조인과 다르게 엔티티 하나만 넘긴다.
    - 일반 조인
        - leftJoin(member.team, team)
        - id가 매칭되는 것을 가져온다.
    - on 조인
        - from(member).leftJoin(team).on()
        - on 조건으로만 필터링 한다.
            - SQL을 보면 id 없이 이름으로만 매칭한다.

```text
t=[Member(id=3, username=member1, age=10), null]
t=[Member(id=4, username=member2, age=20), null]
t=[Member(id=5, username=member3, age=30), null]
t=[Member(id=6, username=member4, age=40), null]
t=[Member(id=7, username=teamA, age=0), Team(id=1, name=teamA)]
t=[Member(id=8, username=teamB, age=0), Team(id=2, name=teamB)]
```

- 이름이 매칭되는 것만 team 데이터를 조인해서 가져왔다.

## fetch join

- fetch join은 SQL이 제공하는건 아니고 SQL 조인을 활용해 한 방에 조회하는 기능이다.
- 성능 최적화에서 주로 사용한다.

### before

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @PersistenceUnit
    EntityManagerFactory emf;

    @Test
    public void fetchJoinNo() throws Exception {
        em.flush();
        em.clear();

        // team이 지연 로딩이기 때문에 데이터가 비어있다.
        Member findMember = queryFactory
                .selectFrom(member)
                .where(member.username.eq("member1"))
                .fetchOne();

        // 해당 데이터가 이미 로딩된 엔티티인지 확인한다.
        boolean loaded = emf.getPersistenceUnitUtil().isLoaded(findMember.getTeam());

        assertThat(loaded).as("페치 조인 미적용").isFalse();
    }
}
```

```sql
select member1
from Member member1
where member1.username = ?1
```

### after

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {
    @Test
    public void fetchJoinUse() throws Exception {
        em.flush();
        em.clear();

        Member findMember = queryFactory
                .selectFrom(member)
                .join(member.team, team)
                // fetch join 적용
                .fetchJoin()
                .where(member.username.eq("member1"))
                .fetchOne();

        // 연관 관계가 같이 로딩된다.
        boolean loaded = emf.getPersistenceUnitUtil().isLoaded(findMember.getTeam());

        assertThat(loaded).as("페치 조인 적용").isTrue();
    }
}
```

```sql
select member0_.member_id as member_i1_1_0_,
       team1_.team_id     as team_id1_2_1_,
       member0_.age       as age2_1_0_,
       member0_.team_id   as team_id4_1_0_,
       member0_.username  as username3_1_0_,
       team1_.name        as name2_2_1_
from member member0_
         inner join
     team team1_ on member0_.team_id = team1_.team_id
where member0_.username = ?
```

- join(), leftJoin() 등 조인 뒤에 fetchJoin()을 추가한다.