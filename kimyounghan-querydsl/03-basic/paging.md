# 페이징

## 조회 건수 제한

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Test
    public void paging1() {
        List<Member> result = queryFactory
                .selectFrom(member)
                .orderBy(member.username.desc())
                // 0부터 시작(zero index)
                .offset(1)
                // 최대 2건 조회
                .limit(2)
                .fetch();

        assertThat(result.size()).isEqualTo(2);
    }
}
```

## 전체 조회 count가 필요하다면

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Test
    public void paging2() {
        QueryResults<Member> queryResults = queryFactory
                .selectFrom(member)
                .orderBy(member.username.desc())
                .offset(1)
                .limit(2)
                .fetchResults();

        assertThat(queryResults.getTotal()).isEqualTo(4);
        assertThat(queryResults.getLimit()).isEqualTo(2);
        assertThat(queryResults.getOffset()).isEqualTo(1);
        assertThat(queryResults.getResults().size()).isEqualTo(2);
    }
}
```

- count 쿼리가 실행되므로 성능에 주의한다.

### 실무에서 페이징 쿼리를 작성할 때 주의점

- 데이터 조회 쿼리는 조인이 필요하지만 count 쿼리는 조인이 필요없는 경우가 있다.
- 자동화 된 count 쿼리는 원본 쿼리처럼 모두 조인을 해버리기 때문에 성능 문제가 있다.
- count 쿼리에 조인이 없도록 성능 최적화가 필요하다면 count 전용 쿼리를 별도로 작성해야 한다.