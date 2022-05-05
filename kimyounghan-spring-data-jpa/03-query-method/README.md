# 쿼리 메서드

{% tabs %} {% tab title="스프링 데이터 JPA" %}

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    List<Member> findByUsernameAndAgeGreaterThan(String username, int age);
}
```

{% endtab %} {% tab title="순수 JPA %}

```java
public class MemberJpaRepository {

    public List<Member> findByUsernameAndAgeGreaterThan(String username, int age) {
        return em.createQuery("select m from Member m where m.username = :username and m.age>:age")
                .setParameter("username", username)
                .setParameter("age", age)
                .getResultList();
    }
}
```

{% endtab %} {% endtabs %}

- 메서드 이름을 분석해서 JPQL을 생성하고 실행한다.
- 엔티티 필드명이 변경되면 메서드 이름도 꼭 함께 변경해야 한다.
    - 그렇지 않으면 애플리케이션 로딩 시점에 오류를 발생시킨다.
- 조회
    - find...By
    - read...By
    - query...By
    - get...By
    - By를 넣지 않으면 조건 없이 전체 조회 한다.
- count
- exists
- 삭제
- distinct
- limit
    - findFirst
    - findFirst3
    - findTop
    - findTop3

[쿼리 메소드 필터 조건](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.query-creation)