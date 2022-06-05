# 동적 쿼리

## BooleanBuilder

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Test
    void dynamicQuery_BooleanBuilder() {
        String usernameParam = "member1";
        Integer ageParam = 10;

        List<Member> result = searchMember1(usernameParam, ageParam);
        Assertions.assertThat(result.size()).isEqualTo(1);
    }

    private List<Member> searchMember1(String usernameCondition, Integer ageCondition) {
        BooleanBuilder builder = new BooleanBuilder();

        // username에 값이 있으면 그 값으로 and 조건을 넣는다.
        if (usernameCondition != null) {
            builder.and(member.username.eq(usernameCondition));
        }

        // age에 값이 있으면 그 값으로 and 조건을 넣는다.
        if (ageCondition != null) {
            builder.and(member.age.eq(ageCondition));
        }

        return queryFactory
                .selectFrom(member)
                .where(builder)
                .fetch();
    }
}
```

```sql
select member0_.member_id as member_i1_1_,
       member0_.age       as age2_1_,
       member0_.team_id   as team_id4_1_,
       member0_.username  as username3_1_
from member member0_
where member0_.username = ?
  and member0_.age = ?
```

- username와 age를 조건으로 쿼리했다.

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Test
    void dynamicQuery_BooleanBuilder() {
        String usernameParam = "member1";
        Integer ageParam = null;

        List<Member> result = searchMember1(usernameParam, ageParam);
        Assertions.assertThat(result.size()).isEqualTo(1);
    }

    ...
}
```

```sql
select member0_.member_id as member_i1_1_,
       member0_.age       as age2_1_,
       member0_.team_id   as team_id4_1_,
       member0_.username  as username3_1_
from member member0_
where member0_.username = ?
```

- null이면 조건으로 넣지 않는다.

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {
    
    ...

    private List<Member> searchMember1(String usernameCondition, Integer ageCondition) {
        // 필수 값을 미리 넣어둘 수도 있다.
        BooleanBuilder builder = new BooleanBuilder(member.username.eq(usernameCondition));

        ...
    }
}
```

- 빌더에 필수 값을 넣어서 초기화할 수도 있다.

## where 다중 파라미터

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Test
    void dynamicQuery_WhereParam() {
        String usernameParam = "member1";
        Integer ageParam = 10;

        List<Member> result = searchMember2(usernameParam, ageParam);
        Assertions.assertThat(result.size()).isEqualTo(1);
    }

    private List<Member> searchMember2(String usernameCond, Integer ageCond) {
        return queryFactory
                .selectFrom(member)
                .where(usernameEq(usernameCond), ageEq(ageCond))
                .fetch();
    }

    private BooleanExpression usernameEq(String usernameCond) {
        return usernameCond != null ? member.username.eq(usernameCond) : null;
    }

    private BooleanExpression ageEq(Integer ageCond) {
        return ageCond != null ? member.age.eq(ageCond) : null;
    }
}
```

```sql
select member0_.member_id as member_i1_1_,
       member0_.age       as age2_1_,
       member0_.team_id   as team_id4_1_,
       member0_.username  as username3_1_
from member member0_
where member0_.username = ?
  and member0_.age = ?
```

- where 조건에서 null 값은 무시된다.
- 메서드를 다른 쿼리에서도 재사용할 수 있다.
- 쿼리 자체의 가독성이 높아진다.

### 조합

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    ...

    private BooleanExpression allEq(String usernameCond, Integer ageCond) {
        return usernameEq(usernameCond).and(ageEq(ageCond));
    }
}
```

```sql
select member0_.member_id as member_i1_1_,
       member0_.age       as age2_1_,
       member0_.team_id   as team_id4_1_,
       member0_.username  as username3_1_
from member member0_
where member0_.username = ?
  and member0_.age = ?
```

- username과 age에 대한 두 조건을 하나로 합쳐서 사용할 수 있다.
- null 처리는 따로 해줘야 한다.