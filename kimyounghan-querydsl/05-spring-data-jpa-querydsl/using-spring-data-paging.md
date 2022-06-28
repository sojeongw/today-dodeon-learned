# 스프링 데이터 페이징 활용

## Querydsl 페이징 연동

```java
public interface MemberRepositoryCustom {
    Page<MemberTeamDto> searchPageSimple(MemberSearchCondition condition, Pageable pageable);

    Page<MemberTeamDto> searchPageComplex(MemberSearchCondition condition, Pageable pageable);
}

```

- 단순한 페이징과 복잡한 페이징을 나눠서 설명할 것이다.

### 단순한 페이징

```java
public class MemberRepositoryImpl implements MemberRepositoryCustom {

    @Override
    public Page<MemberTeamDto> searchPageSimple(MemberSearchCondition condition, Pageable pageable) {
        QueryResults<MemberTeamDto> results = queryFactory
                .select(new QMemberTeamDto(
                        member.id,
                        member.username,
                        member.age,
                        team.id,
                        team.name))
                .from(member)
                .leftJoin(member.team, team)
                .where(usernameEq(condition.getUsername()),
                        teamNameEq(condition.getTeamName()),
                        ageGoe(condition.getAgeGoe()),
                        ageLoe(condition.getAgeLoe()))
                .offset(pageable.getOffset())
                .limit(pageable.getPageSize())
                .fetchResults();

        List<MemberTeamDto> content = results.getResults();
        long total = results.getTotal();

        return new PageImpl<>(content, pageable, total);
    }
}
```

- 전체 카운트를 한번에 조회한다.
    - 실제 DB에 쿼리는 fetch와 count 둘 다 나간다.
    - searchPageSimple()
    - fetchResults()
        - 내용과 전체 카운트를 한번에 조회할 수 있다.
        - 지금은 fetch(), count()로 나뉘었다.

### 복잡한 페이징

```java
public class MemberRepositoryImpl implements MemberRepositoryCustom {

    @Override
    public Page<MemberTeamDto> searchPageComplex(MemberSearchCondition condition, Pageable pageable) {
        List<MemberTeamDto> content = queryFactory
                .select(new QMemberTeamDto(
                        member.id,
                        member.username,
                        member.age,
                        team.id,
                        team.name))
                .from(member)
                .leftJoin(member.team, team)
                .where(usernameEq(condition.getUsername()),
                        teamNameEq(condition.getTeamName()),
                        ageGoe(condition.getAgeGoe()),
                        ageLoe(condition.getAgeLoe()))
                .offset(pageable.getOffset())
                .limit(pageable.getPageSize())
                .fetch();

        long total = queryFactory
                .select(member)
                .from(member)
                .leftJoin(member.team, team)
                .where(usernameEq(condition.getUsername()),
                        teamNameEq(condition.getTeamName()),
                        ageGoe(condition.getAgeGoe()),
                        ageLoe(condition.getAgeLoe()))
                .fetchCount();

        return new PageImpl<>(content, pageable, total);
    }
}
```

- 데이터 내용과 전체 카운트를 별도로 조회한다.
    - 전체 카운트 조회 방법을 최적화 할 수 있다면 유리하다.
        - ex. 전체 카운트 조회 시 조인 쿼리를 줄인다.
- 코드를 리팩터링 해서 내용 쿼리와 전체 카운트 쿼리를 읽기 좋게 분리하면 좋다.

## CountQuery 최적화

- count 쿼리를 생략 할 수 있는 경우
    - 페이지 시작이면서 컨텐츠 사이즈가 페이지 사이즈보다 작을 때
        - 첫번째 페이지가 100개를 출력할 수 있는데 데이터가 3개라면 굳이 count 쿼리를 날릴 필요가 없다.
    - 마지막 페이지 일 때
        - offset + 컨텐츠 사이즈로 전체 사이즈를 구한다.

```java
public class MemberRepositoryImpl implements MemberRepositoryCustom {

    @Override
    public Page<MemberTeamDto> searchPageComplex(MemberSearchCondition condition, Pageable pageable) {
        List<MemberTeamDto> content = queryFactory
                .select(new QMemberTeamDto(
                        member.id.as("memberId"),
                        member.username,
                        member.age,
                        team.id.as("teamId"),
                        team.name.as("teamName")))
                .from(member)
                .leftJoin(member.team, team)
                .where(
                        usernameEq(condition.getUsername()),
                        teamNameEq(condition.getTeamName()),
                        ageGoe(condition.getAgeGoe()),
                        ageLoe(condition.getAgeLoe())
                )
                .offset(pageable.getOffset())
                .limit(pageable.getPageSize())
                .fetch();

        JPAQuery<Long> countQuery = queryFactory
                .select(member.count())
                .from(member)
                .leftJoin(member.team, team)
                .where(usernameEq(condition.getUsername()),
                        teamNameEq(condition.getTeamName()),
                        ageGoe(condition.getAgeGoe()),
                        ageLoe(condition.getAgeLoe()));

        return PageableExecutionUtils.getPage(content, pageable,
                countQuery::fetchOne);
    }
}
```

## 참고

### Querydsl fetchResults(), fetchCount() Deprecated

- fetchResults()는 select를 단순히 count 처리하는 용도로 바꾸는 정도라 복잡한 쿼리에서는 제대로 동작하지 않는다.

```java
public class MemberRepositoryImpl implements MemberRepositoryCustom {

    @Override
    public Page<MemberTeamDto> searchPageSimple(MemberSearchCondition condition, Pageable pageable) {
        Long totalCount = queryFactory
                // .select(Wildcard.count) // select count(*)
                .select(member.count()) // select count(member.id)
                .from(member)
                .fetchOne();
    }
}
```

- select(Wildcard.count)
    - count(*)을 사용하고 싶을 때 사용한다.
- select(member.count())
    - select count(member.id)로 처리한다.
- fetchOne()
    - 응답 결과는 숫자 하나이므로 fetchOne()을 사용한다.

## CountQuery 최적화

- count 쿼리가 생략 가능한 경우 생략해서 처리한다.
    - 페이지 시작점이면서 컨텐츠 사이즈가 페이지 사이즈보다 작을 때
    - 마지막 페이지일 때
        - offset + 컨텐츠 사이즈를 더해 전체 사이즈를 구한다.

```java
public class MemberRepositoryImpl implements MemberRepositoryCustom {

    @Override
    public Page<MemberTeamDto> searchPageSimple(MemberSearchCondition condition, Pageable pageable) {
        
        ...

        JPAQuery<Member> countQuery = queryFactory
                .select(member)
                .from(member)
                .leftJoin(member.team, team)
                .where(usernameEq(condition.getUsername()),
                        teamNameEq(condition.getTeamName()),
                        ageGoe(condition.getAgeGoe()),
                        ageLoe(condition.getAgeLoe()));

        return PageableExecutionUtils.getPage(content, pageable, countQuery::fetchCount);
    }
}
```

- count 쿼리는 fetchCount()를 하지 않으면 실제 쿼리가 날아가지 않는다.
- 인자로 함수를 넘기면 content, pageable의 토탈 사이즈를 보고 실행한다.

## 컨트롤러 개발

- 지금까지 한 내용이 실제 동작하는지 확인한다.

```java

@RestController
@RequiredArgsConstructor
public class MemberController {
    private final MemberJpaRepository memberJpaRepository;
    private final MemberRepository memberRepository;

    @GetMapping("/v1/members")
    public List<MemberTeamDto> searchMemberV1(MemberSearchCondition condition) {
        return memberJpaRepository.search(condition);
    }

    @GetMapping("/v2/members")
    public Page<MemberTeamDto> searchMemberV2(MemberSearchCondition condition,
                                              Pageable pageable) {
        return memberRepository.searchPageSimple(condition, pageable);
    }

    @GetMapping("/v3/members")
    public Page<MemberTeamDto> searchMemberV3(MemberSearchCondition condition,
                                              Pageable pageable) {
        return memberRepository.searchPageComplex(condition, pageable);
    }
}
```

```text
http://localhost:8080/v2/members?size=5&page=2
```

```sql
select count(member1)
from Member member1
         left join member1.team as team
```

- count 쿼리가 나간다.

```text
http://localhost:8080/v3/members?size=110&page=0
```

```sql
select member1.id as memberId, member1.username, member1.age, team.id as teamId, team.name as teamName
from Member member1
         left join member1.team as team
```

- select 쿼리만 나간다.
- 두 번째 페이지로 넘어갈 컨텐츠 개수가 없기 때문에 count 쿼리를 날리지 않는다.

## 정렬

- 스프링 데이터 JPA가 OrderSpecifier로 정렬 기능을 제공한다.

```java
public class MemberRepositoryImpl implements MemberRepositoryCustom {

    @Override
    public Page<MemberTeamDto> searchPageSimple(MemberSearchCondition condition, Pageable pageable) {

        ...

        JPAQuery<Member> query = queryFactory
                .selectFrom(member);
        
        for (Sort.Order o : pageable.getSort()) {
            PathBuilder pathBuilder = new PathBuilder(member.getType(), member.getMetadata());
            
            query.orderBy(new OrderSpecifier(o.isAscending() ? Order.ASC : Order.DESC,
                    pathBuilder.get(o.getProperty())));
            
            List<Member> result = query.fetch();
        }
    }
```

- 스프링 데이터 JPA의 정렬을 Querydsl의 정렬로 직접 변환할 수 있다.

### 참고

- 정렬은 조건이 조금만 복잡해져도 Pageable의 Sort를 쓰기 어렵다.
- 루트 엔티티 범위를 넘어서는 동적 정렬이 필요하면 Sort보다는 파라미터를 받아서 직접 처리하자.