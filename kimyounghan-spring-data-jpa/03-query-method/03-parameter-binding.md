# 파라미터 바인딩

## 위치 기반

```sql
select m
from Member m
where m.username = ?0
```

- 거의 사용하지 않는다.
    - 위치가 바뀌면 버그가 발생한다.

## 이름 기반

```sql
select m
from Member m
where m.username = :name
```

{% tabs %} {% tab title="MemberRepository.java" %}

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @Query("select m from Member m where m.username = :name")
    Member findMembers(@Param("name") String username);

    // 컬렉션 파라미터 바인딩
    @Query("select m from Member m where m.username in :names")
    List<Member> findByNames(@Param("names") List<String> names);
}
```

{% endtab %} {% endtabs %}

- in 절을 사용한 Collection 타입을 지원한다.