# SQL Function 호출하기

- JPA 등 Dialect에 등록된 내용만 호출할 수 있다.

## 컬럼명 변경

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {
    @Test
    void sqlFunction() {
        String result = queryFactory
                .select(Expressions.stringTemplate(
                        "function('replace', {0}, {1}, {2})",
                        member.username, "member", "M"
                )).from(member)
                .fetchFirst();
    }
}
```

```sql
select replace(member0_.username, ?, ?) as col_0_0_
from member member0_ limit ?
select replace(member0_.username, 1, NULL) as col_0_0_
from member member0_ limit ?;
```

- member라는 단어를 모두 m으로 변경해서 조회한다.

## 소문자 변경

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {
    @Test
    void sqlFunction() {
        String result = queryFactory
                .select(member.username)
                .from(member)
                .where(member.username.eq(
                        Expressions.stringTemplate("function('lower', {0})",
                                member.username)))
                .from(member)
                .fetchFirst();
    }
}
```

```sql
select member0_.username as col_0_0_ from member member0_ where member0_.username=lower(member0_.username) limit ?
select member0_.username as col_0_0_ from member member0_ where member0_.username=lower(member0_.username) limit 1;
```

- 소문자 변환 등 자주 사용하는 일반적인 기능은 ANSI 표준이라서 기본적으로 내장되어 있다.

```text
.where(member.username.eq(member.username.lower()))
```

- 이렇게 lower()로 대체할 수 있다.