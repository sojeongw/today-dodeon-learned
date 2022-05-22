# Native Query

- SQL을 직접 짜는 방식
- 정말 어쩔 수 없을 때만 사용한다.
    - 최근에는 스프링 데이터 Projection을 활용한다.

## 스프링 데이터 JPA 기반 네이티브 쿼리

- 페이징 지원
- 반환 타입
    - Object[]
    - Tuple
    - DTO
        - 스프링 데이터 인터페이스인 Projections 지원

{% tabs %} {% tab title="MemberRepository.java" %}

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @Query(value = "select * from member where username = ?", nativeQuery = true)
    Member findByNativeQuery(String username);

}
```

{% endtab %} {% tab title="MemberRepositoryTest.java" %}

```java
class MemberRepositoryTest {

    @Test
    void nativeQuery() {
        Team teamA = new Team("teamA");
        em.persist(teamA);

        Member m1 = new Member("m1", 0, teamA);
        Member m2 = new Member("m2", 0, teamA);

        em.persist(m1);
        em.persist(m2);

        em.flush();
        em.clear();

        Member result = memberRepository.findByNativeQuery("m1");
    }
}
```

{% endtab %} {% endtabs %}

```sql
select *
from member
where username = 'm1';
```

- 조회하려는 필드를 select 절에 다 적어줘야 해서 불편하다.
- JPQL처럼 애플리케이션 로딩 시점에 문법을 확인하는 것이 불가하다.
- 동적 쿼리를 쓸 수 없다.
- 반환 타입이 제한되어 있다.
- Sort 파라미터를 통한 정렬이 정상 동작 하지 않을 수도 있다.
    - 믿지 말고 직접 처리하자.
- JPQL은 위치 기반 파라미터를 1부터 시작하지만 SQL은 0부터 시작한다.

## Projections 활용

- 스프링 데이터 JPA 네이티브 쿼리와 인터페이스 기반 Projections를 활용할 수 있다.

{% tabs %} {% tab title="MemberProjection.java" %}

```java
public interface MemberProjection {

    Long getId();

    String getUsername();

    String getTeamUsername();

}

```

{% endtab %} {% tab title="MemberRepository.java" %}

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @Query(value = "SELECT m.member_id as id, m.username, t.name as teamName " +
            "FROM member m left join team t",
            countQuery = "SELECT count(*) from member",
            nativeQuery = true)
    Page<MemberProjection> findByNativeProjection(Pageable pageable);

}
```

{% endtab %} {% tab title="MemberRepositoryTest.java" %}

```java
class MemberRepositoryTest {

    @Test
    void nativeQuery() {
        ...

        // 인덱스가 0부터 시작하므로 0으로 조회한다.
        Page<MemberProjection> result = memberRepository.findByNativeProjection(PageRequest.of(0, 10));
    }
}
```

{% endtab %} {% endtabs %}

```sql
SELECT m.member_id as id, m.username, t.name as teamName
FROM member m
         left join team t limit 10;

SELECT count(*)
from member;
```

## 동적 네이티브 쿼리

```java
class Example {
    public static void main(String[] args) {
        String sql = "select m.username as username from member m";

        List<MemberDto> result = em.createNativeQuery(sql)
                .setFirstResult(0)
                .setMaxResults(10)
                .unwrap(NativeQuery.class)
                .addScalar("username")
                .setResultTransformer(Transformers.aliasToBean(MemberDto.class))
                .getResultList();
    }
}
```

- 하이버네이트를 직접 활용한다.
- 스프링 JdbcTemplate, myBatis, jooq 같은 외부 라이브러리를 사용하는걸 추천한다.

