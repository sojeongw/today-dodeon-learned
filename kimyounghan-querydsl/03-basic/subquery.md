# 서브 쿼리

- com.querydsl.jpa.JPAExpressions를 사용한다.

## eq

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Test
    public void subQuery() throws Exception {
        // 밖에 있는 member와 서브 쿼리의 member의 alias가 겹치면 안되므로 직접 만든다.
        QMember memberSub = new QMember("memberSub");

        List<Member> result = queryFactory
                .selectFrom(member)
                .where(member.age.eq(
                        JPAExpressions
                                .select(memberSub.age.max())
                                .from(memberSub)))
                .fetch();

        assertThat(result).extracting("age").containsExactly(40);
    }
}
```

```sql
select member0_.member_id as member_i1_1_,
       member0_.age       as age2_1_,
       member0_.team_id   as team_id4_1_,
       member0_.username  as username3_1_
from member member0_
where member0_.age = (
    select max(member1_.age)
    from member member1_
)
```

- 나이가 가장 많은 회원 조회

## goe

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Test
    public void subQueryGoe() throws Exception {
        QMember memberSub = new QMember("memberSub");

        List<Member> result = queryFactory
                .selectFrom(member)
                .where(member.age.goe(
                        JPAExpressions
                                .select(memberSub.age.avg())
                                .from(memberSub)))
                .fetch();

        assertThat(result).extracting("age")
                .containsExactly(30, 40);
    }
}
```

```sql
select member0_.member_id as member_i1_1_,
       member0_.age       as age2_1_,
       member0_.team_id   as team_id4_1_,
       member0_.username  as username3_1_
from member member0_
where member0_.age >= (
    select avg(cast(member1_.age as double))
    from member member1_
)
```

- 나이가 평균 이상인 회원 조회

## in

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Test
    public void subQueryIn() throws Exception {
        QMember memberSub = new QMember("memberSub");

        List<Member> result = queryFactory
                .selectFrom(member)
                .where(member.age.in(
                        JPAExpressions
                                .select(memberSub.age)
                                .from(memberSub)
                                .where(memberSub.age.gt(10))))
                .fetch();

        assertThat(result).extracting("age")
                .containsExactly(20, 30, 40);
    }
}
```

```sql
select member0_.member_id as member_i1_1_,
       member0_.age       as age2_1_,
       member0_.team_id   as team_id4_1_,
       member0_.username  as username3_1_
from member member0_
where member0_.age in (
    select member1_.age
    from member member1_
    where member1_.age > ?
)
```

## select절에 subquery

```java

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Test
    void selectSubQuery() {
        QMember memberSub = new QMember("memberSub");

        List<Tuple> fetch = queryFactory
                .select(member.username,
                        JPAExpressions
                                .select(memberSub.age.avg())
                                .from(memberSub)
                ).from(member)
                .fetch();

        for (Tuple tuple : fetch) {
            System.out.println("username = " + tuple.get(member.username));
            System.out.println("age = " +
                    tuple.get(JPAExpressions.select(memberSub.age.avg())
                            .from(memberSub)));
        }
    }
}
```

```sql
select member0_.username      as col_0_0_,
       (select avg(cast(member1_.age as double))
        from member member1_) as col_1_0_
from member member0_
```

## static import 활용

```java
import static com.querydsl.jpa.JPAExpressions.select;

@SpringBootTest
@Transactional
public class QuerydslBasicTest {

    @Test
    void selectSubQuery() {
        List<Member> result = queryFactory
                .selectFrom(member)
                .where(member.age.eq(
                        select(memberSub.age.max())
                                .from(memberSub)
                )).fetch();

        ...
    }
}
```

- JPAExpressions 없이 깔끔하게 정리할 수 있다.

## from절의 서브쿼리 한계

- JPA 서브 쿼리는 from절에 적용할 수 없다.
- 따라서 Querydsl도 지원하지 않는다.
- 원래 select로 안되는데 하이버네이트 구현체를 사용할 때는 가능하다.
    - Querydsl도 하이버네이트 구현체를 쓰기 때문에 가능하다.

### 해결 방안

- 서브 쿼리는 join으로 변경한다.
    - 불가능할 때도 있다.
- 애플리케이션에서 쿼리를 2번 분리해 실행한다.
- nativeSQL을 사용한다.

## 참고

대부분 화면에 맞추려고 sql에서 데이터를 정제하다 보니 복잡해지는 문제이기 때문에 애플리케이션 로직과 뷰에서 풀도록 한다. DB는 데이터를 퍼올리는 용도일 뿐이다. 최소한의 정제만 해서 가져오도록 하자.

또한, 실시간 트래픽이 중요할 경우엔 한 방 쿼리가 중요할 수 있지만 어드민 같은 트래픽이 몰리지 않는 곳에서 굳이 애쓰지 말자. 그냥 쿼리를 여러번 날려서 순차적으로 해결하자. 