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