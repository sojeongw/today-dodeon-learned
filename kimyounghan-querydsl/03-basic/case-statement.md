# Case 문

- select, where, order by에서 사용 가능하다.

## 단순한 조건

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Test
    void caseStatement() {
        List<String> result = queryFactory
                .select(member.age
                        .when(10).then("열살")
                        .when(20).then("스무살")
                        .otherwise("기타"))
                .from(member)
                .fetch();
    }
}
```

```sql
select case
           when member0_.age = ? then ?
           when member0_.age = ? then ?
           else '기타'
           end as col_0_0_
from member member0_
```

## 복잡한 조건

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Test
    void caseStatement() {
        List<String> result = queryFactory
                .select(new CaseBuilder()
                        .when(member.age.between(0, 20)).then("0~20살")
                        .when(member.age.between(21, 30)).then("21~30살")
                        .otherwise("기타"))
                .from(member)
                .fetch();
    }
}
```

```sql
select case
           when member0_.age between ? and ? then ?
           when member0_.age between ? and ? then ?
           else '기타'
           end as col_0_0_
from member member0_
```

- 복잡한 조건에는 CaseBuilder를 사용한다.

## orderBy + case

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Test
    void orderby_case() {
        // 복잡한 조건은 rankPath처럼 변수로 선언해서 활용한다.
        NumberExpression<Integer> rankPath = new CaseBuilder()
                .when(member.age.between(0, 20)).then(2)
                .when(member.age.between(21, 30)).then(1)
                .otherwise(3);

        List<Tuple> result = queryFactory
                .select(member.username, member.age, rankPath)
                .from(member)
                .orderBy(rankPath.desc())
                .fetch();

        for (Tuple tuple : result) {
            String username = tuple.get(member.username);
            Integer age = tuple.get(member.age);
            Integer rank = tuple.get(rankPath);

            System.out.println("username = " + username + " age = " + age + " rank = " + rank);
        }
    }
}
```

```sql
select member0_.username as col_0_0_,
       member0_.age      as col_1_0_,
       case
           when member0_.age between ? and ? then ?
           when member0_.age between ? and ? then ?
           else 3
           end           as col_2_0_
from member member0_
order by case
             when member0_.age between ? and ? then ?
             when member0_.age between ? and ? then ?
             else 3
             end desc
```

```text
username = member4 age = 40 rank = 3
username = member1 age = 10 rank = 2
username = member2 age = 20 rank = 2
username = member3 age = 30 rank = 1
```

- 복잡한 조건은 변수로 선언해 select, orderBy에서 활용한다.
- 웬만하면 DB에서는 최소한의 조건만 걸고 case로 들어갈 수 있는 조건들은 애플리케이션에서 하자.