# 스프링 데이터 JPA와 Querydsl

## 스프링 데이터 JPA 리포지토리로 변경

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    List<Member> findByUsername(String username);
}
```

- findByUsername()처럼 Spring Data JPA가 자동으로 제공하지 않는 기능은 직접 선언한다.

## 사용자 정의 리포지토리

Querydsl을 쓰려면 구현 코드를 만들어야 하는데 스프링 데이터 JPA는 인터페이스로 동작하기 때문에 사용자 정의 리포지토리가 필요하다.

![](../../.gitbook/assets/kimyounghan-querydsl/05/스크린샷%202022-07-23%20오후%207.58.28.png)

1. 사용자 정의 인터페이스 작성
2. 사용자 정의 인터페이스 구현
3. 스프링 데이터 리포지토리에 사용자 정의 인터페이스 상속

### 1. 사용자 정의 인터페이스 작성

```java
public interface MemberRepositoryCustom {
    List<MemberTeamDto> search(MemberSearchCondition condition);
}

```

- 인터페이스 이름은 자유롭게 지어도 된다.

### 2. 사용자 정의 인터페이스 구현

```java
public class MemberRepositoryImpl implements MemberRepositoryCustom {
    private final JPAQueryFactory queryFactory;

    public MemberRepositoryImpl(EntityManager em) {
        this.queryFactory = new JPAQueryFactory(em);
    }

    @Override
    public List<MemberTeamDto> search(MemberSearchCondition condition) {
        return queryFactory
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
                .fetch();
    }

    private BooleanExpression usernameEq(String username) {
        return hasText(username) ? member.username.eq(username) : null;
    }

    private BooleanExpression teamNameEq(String teamName) {
        return hasText(teamName) ? team.name.eq(teamName) : null;
    }

    private BooleanExpression ageGoe(Integer ageGoe) {
        return ageGoe == null ? member.age.goe(ageGoe) : null;
    }

    private BooleanExpression ageLoe(Integer ageLoe) {
        return ageLoe == null ? member.age.loe(ageLoe) : null;
    }
}
```

- 반드시 사용자 정의 인터페이스 이름 + `Impl`로 만들어야 한다.

### 3. 스프링 데이터 리포지토리에 사용자 정의 인터페이스 상속

```java
public interface MemberRepository extends JpaRepository<Member, Long>, MemberRepositoryCustom {
    List<Member> findByUsername(String username);
}
```

```sql
select member0_.member_id as col_0_0_,
       member0_.username  as col_1_0_,
       member0_.age       as col_2_0_,
       team1_.team_id     as col_3_0_,
       team1_.name        as col_4_0_
from member member0_
         left outer join team team1_ on member0_.team_id = team1_.team_id
where team1_.name = ?
```