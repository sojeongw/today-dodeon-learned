# 결과 조회

- fetch()
    - 리스트 조회
    - 데이터가 없으면 빈 리스트 반환
- fetchOne()
    - 단건 조회
    - 결과가 없으면
        - null
    - 결과가 둘 이상이면
        - com.querydsl.core.NonUniqueResultException
- fetchFirst()
    - limit(1).fetchOne()
- fetchResults()
    - 페이징 정보 포함
    - total count 쿼리 추가 실행
- fetchCount()
    - count 쿼리로 변경해서 count 수 조회

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Test
    void result() {
        // List
        List<Member> fetch = queryFactory
                .selectFrom(member)
                .fetch();

        // 단건
        Member findMember1 = queryFactory
                .selectFrom(member)
                .fetchOne();

        // 맨 처음 한 건만 조회
        Member findMember2 = queryFactory
                .selectFrom(member)
                .fetchFirst();

        // 페이징에서 사용
        QueryResults<Member> results = queryFactory
                .selectFrom(member)
                .fetchResults();

        // count 쿼리로 변경
        long count = queryFactory
                .selectFrom(member)
                .fetchCount();
    }
}
```

- 스프링부트 2.6 이상에서는 fetchResults(), fetchCount()가 deprecated 되었다.