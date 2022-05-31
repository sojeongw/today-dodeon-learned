# 집합 함수

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Test
    public void aggregation() throws Exception {
        List<Tuple> result = queryFactory
                .select(member.count(),
                        member.age.sum(),
                        member.age.avg(),
                        member.age.max(),
                        member.age.min())
                .from(member)
                .fetch();

        Tuple tuple = result.get(0);

        assertThat(tuple.get(member.count())).isEqualTo(4);
        assertThat(tuple.get(member.age.sum())).isEqualTo(100);
        assertThat(tuple.get(member.age.avg())).isEqualTo(25);
        assertThat(tuple.get(member.age.max())).isEqualTo(40);
        assertThat(tuple.get(member.age.min())).isEqualTo(10);
    }
}
```

- JPQL이 제공하는 모든 집합 함수를 제공한다.
- Tuple
    - 프로젝션과 결과 반환 챕터에서 설명할 예정이다.

## GroupBy

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Test
    public void group() throws Exception {
        List<Tuple> result = queryFactory
                .select(team.name, member.age.avg())
                .from(member)
                .join(member.team, team)
                .groupBy(team.name)
                .fetch();

        Tuple teamA = result.get(0);
        Tuple teamB = result.get(1);

        assertThat(teamA.get(team.name)).isEqualTo("teamA");
        assertThat(teamA.get(member.age.avg())).isEqualTo(15);

        assertThat(teamB.get(team.name)).isEqualTo("teamB");
        assertThat(teamB.get(member.age.avg())).isEqualTo(35);
    }
}
```

```sql
select team.name, avg(member1.age)
from Member member1
         inner join member1.team as team
group by team.name
```

- 각 팀의 이름으로 묶는다.

## Having

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Test
    public void group() throws Exception {
        List<Tuple> result = queryFactory
                ...
                .groupBy(item.price)
                .having(item.price.gt(1000));
    }
}
```

- groupby한 결과 중에서 having 조건에 해당하는 것만 뽑는다.