# Projections

- 엔티티 대신 DTO로 편리하게 조회할 때 사용한다.
    - ex. 전체 엔티티가 아니라 회원 이름만 조회하고 싶을 때

## 인터페이스 기반 Closed Projections

{% tabs %} {% tab title="UsernameOnly.java" %}

```java
public interface UsernameOnly {

    // 조회할 엔티티 필드를 getter 형식으로 지정하면 해당 필드만 선택해서 조회한다.
    String getUsername();
}
```

{% endtab %} {% tab title="MemberRepository.java" %}

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    // 메서드 이름은 자유롭게 하면 되고 반환 타입으로 인지한다.
    List<UsernameOnly> findProjectionsByUsername(String username);

}
```

{% endtab %} {% tab title="MemberRepositoryTest.java" %}

```java
class MemberRepositoryTest {

    @Test
    public void projections() throws Exception {
        Team teamA = new Team("teamA");
        em.persist(teamA);

        Member m1 = new Member("m1", 0, teamA);
        Member m2 = new Member("m2", 0, teamA);

        em.persist(m1);
        em.persist(m2);

        em.flush();
        em.clear();

        List<UsernameOnly> result = memberRepository.findProjectionsByUsername("m1");

        Assertions.assertThat(result.size()).isEqualTo(1);
    }
}
```

{% endtab %} {% endtabs %}

```sql
select m.username
from member m
where m.username = 'm1';
```

- 원하는 필드로 쿼리가 잘 나간다.

```java
public interface UsernameOnly {

    // 조회할 엔티티 필드를 getter 형식으로 지정하면 해당 필드만 선택해서 조회한다.
    String getUsername();
}
```

- getter 즉, 프로퍼티 형식의 인터페이스를 제공하면 스프링 데이터 JPA가 구현체를 알아서 제공한다.

## 인터페이스 기반 Open Projections

```java
import org.springframework.beans.factory.annotation.Value;

public interface UsernameOnly {

    @Value("#{target.username + ' ' + target.age + ' ' + target.team.name}")
    String getUsername();
}
```

```sql
select member0_.member_id          as member_i1_1_,
       member0_.created_by         as created_2_1_,
       member0_.created_date       as created_3_1_,
       member0_.last_modified_by   as last_mod4_1_,
       member0_.last_modified_date as last_mod5_1_,
       member0_.age                as age6_1_,
       member0_.team_id            as team_id8_1_,
       member0_.username           as username7_1_
from member member0_
where member0_.username = 'm1';
```

- SpEL 문법을 지원한다.
- DB에서 엔티티 필드를 다 조회한 다음에 계산하기 때문에 select 절 최적화가 안된다.

## 클래스 기반 Projection

{% tabs %} {% tab title="UsernameOnlyDto.java" %}

```java
public class UsernameOnlyDto {

    private final String username;

    // 생성자의 파라미터 이름으로 매칭한다.
    public UsernameOnlyDto(String username) {
        this.username = username;
    }

    public String getUsername() {
        return username;
    }
}
```

{% endtab %} {% tab title=".java" %}

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    List<UsernameOnlyDto> findProjectionsByUsername(String username);

}
```

{% endtab %} {% endtabs %}

```sql
select member0_.username as col_0_0_
from member member0_
where member0_.username = 'm1';
```

- 인터페이스 대신 DTO 형식도 가능하다.
- 생성자의 파라미터 이름으로 매칭한다.

## 동적 Projections

{% tabs %} {% tab title="MemberRepository.java" %}

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    <T> List<T> findProjectionsByUsername(String username, Class<T> type);

}
```

{% endtab %} {% tab title="MemberRepositoryTest.java" %}

```java
class MemberRepositoryTest {

    @Test
    public void projections() throws Exception {
        
        ...

        List<UsernameOnly> result = memberRepository.findProjectionsByUsername("m1", UsernameOnly.class);
    }
}
```

{% endtab %} {% endtabs %}

- 제네릭 타입을 통해 동적으로 프로젝션 데이터를 변경할 수 있다.

## 중첩 구조 처리

{% tabs %} {% tab title="NestedClosedProjection.java" %}

```java
public interface NestedClosedProjection {

    String getUsername();

    TeamInfo getTeam();

    interface TeamInfo {
        String getName();
    }

}

```

{% endtab %} {% tab title=".java" %}

```java
class MemberRepositoryTest {

    @Test
    public void projections() throws Exception {

        ...

        List<UsernameOnly> result = memberRepository.findProjectionsByUsername("m1", UsernameOnly.class);
    }
}
```

{% endtab %} {% endtabs %}

```sql
select member0_.username as col_0_0_, team1_.team_id as col_1_0_, team1_.team_id as team_id1_2_, team1_.name as name2_2_
from member member0_
         left outer join team team1_ on member0_.team_id = team1_.team_id
where member0_.username = 'm1';
```

- 회원과 더불어 연관된 팀을 가져온다.
- 프로젝션 대상이 root 엔티티면 JPQL select 절을 최적화 한다.
    - username처럼 처음에 나오는 루트는 최적화 해서 필요한 데이터만 불러온다.
- 프로젝션 대상이 root가 아니면 left outer join으로 처리한다.
    - 뒤에 있는 team은 엔티티를 다 가지고 온다.

### 정리

- 프로젝션 대상이 root 엔티티면 유용하다.
- 실무의 복잡한 쿼리를 해결하기엔 한계가 있다.
- 실무에서는 단순할 때만 사용하고 조금이라도 복잡해지면 QueryDSL을 사용한다.