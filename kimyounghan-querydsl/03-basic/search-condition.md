# 검색 조건 쿼리

{% tabs %} {% tab title="QuerydslBasicTest.java" %}

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Test
    void search() {
        Member findMember = queryFactory
                // select와 from이 같으면 합칠 수 있다.
                .selectFrom(member)
                .where(member.username.eq("member1")
                        .and(member.age.eq(10))
                ).fetchOne();

        assertThat(findMember.getUsername()).isEqualTo("member1");
    }
}
```

{% endtab %} {% endtabs %}

```sql
select member1
from Member member1
where member1.username = ?1
  and member1.age = ?2
```

- and, or를 메서드 체인으로 연결할 수 있다.
- select, from을 selectFrom으로 합칠 수 있다.

## 제공하는 검색 조건

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    void example() {
        member.username.eq("member1")

        // username != 'member1'
        member.username.ne("member1")

        // username != 'member1'
        member.username.eq("member1").not()

        // 이름이 is not null
        member.username.isNotNull()

        // age in (10,20)
        member.age.in(10, 20)

        // age not in (10, 20)
        member.age.notIn(10, 20)

        // between 10, 30
        member.age.between(10, 30)

        // age >= 30
        member.age.goe(30)

        // age > 30        
        member.age.gt(30)

        // age <= 30
        member.age.loe(30)

        // age < 30        
        member.age.lt(30)

        // like 검색         
        member.username.like("member%")

        // like ‘%member%’ 검색
        member.username.contains("member")

        // like ‘member%’ 검색
        member.username.startsWith("member")
    }
}
```

- JPQL이 제공하는 모든 검색 조건을 제공한다.

## AND 조건을 파라미터로 처리

{% tabs %} {% tab title="QuerydslBasicTest.java" %}

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Test
    public void searchAndParam() {
        List<Member> result1 = queryFactory
                .selectFrom(member)
                .where(member.username.eq("member1"),
                        member.age.eq(10))
                .fetch();

        assertThat(result1.size()).isEqualTo(1);
    }
}
```

{% endtab %} {% endtabs %}

- where()의 파라미터로 검색 조건을 추가하면 and가 적용된다.
- null 값을 무시하기 때문에 메서드 추출을 활용해 동적 쿼리를 깔끔하게 만들 수 있다.