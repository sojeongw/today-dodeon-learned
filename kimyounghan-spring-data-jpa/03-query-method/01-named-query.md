# JPA NamedQuery

{% tabs %} {% tab title="스프링 데이터 JPA" %}

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    //    @Query(name = "Member.findByUsername") 생략 가능
    List<Member> findByUsername(@Param("username") String username);
}
```

{% endtab %} {% tab title="순수 JPA" %}

```java
public class MemberRepository {
    public List<Member> findByUsername(String username) {
        
        ...

        List<Member> resultList =
                em.createNamedQuery("Member.findByUsername", Member.class)
                        .setParameter("username", username)
                        .getResultList();
    }
} 
```

{% endtab %} {% tab title="NamedQuery 정의" %}

```java

@Entity
@NamedQuery(
        name = "Member.findByUsername",
        query = "select m from Member m where m.username = :username")
public class Member {
    ...
}
```

{% endtab %}{% endtabs %}

- 쿼리의 이름을 정해놓고 불러올 수 있는 기능
- @Query를 생략하고 메서드만으로도 호출할 수 있다.
    - `엔티티 클래스.메서드 이름`으로 NamedQuery가 선언된 게 있는지 찾아준다.
    - ex. Member.findByUsername
    - 없다면 메서드 이름으로 쿼리를 자동 생성한다.
- 애플리케이션 로딩 시점에 미리 SQL를 만들어보고 문법 오류를 체크해준다.
- NamedQuery 기능은 실무에서 잘 사용하지 않는다.
    - 대신 @Query를 사용해 직접 정의한다.
