# fetch join

- SQL 조인 종류는 아니다.
- JPQL에서 성능 최적화를 위해 제공하는 전용 기능이다.
- 연관된 Entity나 컬렉션을 SQL 한 번에 조회할 수 있다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%202.27.05.png)

- `select m`만 썼는데도 회원을 조회하면서 연관된 팀도 SQL에서 한 번에 조회한다.
    - SQL을 보면 회원 `M.*`과 팀 `T.*`을 함께 select 한다.
- 즉시 로딩으로 가져오는 방법과 똑같다.
    - 단지 join fetch라고 명시적으로 선언해서 원하는 객체 그래프를 한 번에 조회하는 것이다.

## 단일 값 fetch join

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%202.31.43.png)

- 회원 1, 2는 팀 A 소속이다.
- 회원 3은 팀 B 소속이다.
- 회원 4는 소속 팀이 없다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%202.31.53.png)

- fetch join으로 회원과 팀을 한 번에 조회할 수 있다.
- 총 5개의 엔티티를 1차 캐시에 저장해놓는다.
    - 회원 1, 2, 3, 팀 A, 팀 B

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%202.31.49.png)

- 소속 팀이 없는 회원은 제외되는 inner join이다.

### 일반 join 예시

```java
public class JpaMain {

    public static void main(String[] args) {
        String jpql = "select m from Member m";
        List<Member> members = em.createQuery(jpql, Member.class)
                .getResultList();

        for (Member member : members) {
            System.out.println("member = " + member.getUsername() + ", "
                    + "teamName = " + member.getTeam().name());
        }
    }
}
```

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%202.37.24.png)

- 연관 관계에 있는 team이 프록시로 들어왔다가, `getTeam().getName()` 을 호출할 때 지연 로딩으로 select 쿼리가 나간다.
    - 회원 1이 팀 A를 불러 올 때 최초 SQL을 날린다.
    - 회원 2는 같은 팀 A이므로 1차 캐시에서 가져온다.
    - 회원 3의 팀 B는 영속성 컨텍스트에 존재하지 않으므로 새로운 쿼리를 날린다.
- N + 1 문제
    - 소속이 다 다르면 N개만큼 계속 쿼리가 나가야 한다.
    - 회원을 가져오기 위해 최초에 날린 1번의 쿼리 + 회원이 소속된 팀 개수 N만큼 날린다.

### fetch join 예시

```java
public class JpaMain {

    public static void main(String[] args) {
        // fetch join 사용
        String jpql = "select m from Member m join fetch m.team";
        List<Member> members = em.createQuery(jpql, Member.class)
                .getResultList();

        for (Member member : members) {
            System.out.println("username = " + member.getUsername() + ", "
                    + "teamName = " + member.getTeam().name());
        }
    }
}
```

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%202.43.33.png)

- fetch join으로 회원과 팀을 한 번에 조회한다.
    - 지연 로딩하지 않는다.
    - 즉, `member.getTeam().name()` 호출 시에 프록시가 아니라 실제 Entity가 담겨있다.
- 지연 로딩으로 되어있어도 fetch join이 우선 순위를 가진다.

## 컬렉션 fetch join

- 일대다 관계에서의 컬렉션 조회

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%202.47.42.png)

- 반대로 팀에서 N인 회원을 조회한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%202.50.49.png)

- 회원만큼 팀이 중복으로 출력된다.
- DB 입장에서는 일대다 join 시 데이터가 뻥튀기 된다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%202.48.36.png)

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%202.48.40.png)

- 팀 A로 조인을 하면 회원이 2명이므로 데이터를 2개 만들어 낸다.
- JPA 입장에서는 회원이 몇 명 있는지 미리 알 수가 없으니 DB가 반환하는 대로 가져와야 한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%202.48.45.png)

- row는 2개지만 영속성 컨텍스트에 있는 ID는 같이 공유하므로 주소 값이 같다.

## fetch join과 DISTINCT

- SQL의 DISTINCT
    - 중복을 완벽하게 제거할 수 없다.
- JPQL은 DISTINCT에 대해 2가지 기능을 제공한다.
    - SQL에 DISTINCT를 추가
    - SQL 결과를 애플리케이션에서 받은 뒤 엔티티의 중복을 제거

```sql
select distinct t
from Team t
         join fetch t.members
where t.name = '팀A'
```

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%203.03.07.png)

- SQL의 DISTINCT는 데이터가 100% 똑같아야 적용된다.
    - join 결과를 보면 각 row의 PK가 다르기 때문에 중복 제거에 실패한다.
- 그래서 JPA가 애플리케이션에서 추가적인 중복 제거를 시도한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%203.05.25.png)

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%203.06.47.png)

- JPQL의 DISTINCT가 같은 식별자를 가진 Team Entity를 자동으로 제거한다.

## 일반 join과의 차이

### 일반 join

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%203.09.15.png)

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%203.11.03.png)

- 연관된 entity를 함께 조회하지 않는다.
    - jpql에 일반 join을 쓰면 sql에서 team만 select 한다.
        - Team Entity만 조회하고 Member Entity는 조회하지 않는다.
    - `team.getMembers()` 하는 시점에 쿼리가 다시 나간다.

### fetch join

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%203.13.12.png)

- 팀 정보를 가져올 때 회원 정보도 함께 가져온다.
- fetch join을 사용할 때만 연관된 Entity를 함께 조회한다.
    - 즉시 로딩의 개념
    - 객체 그래프를 SQL 한 번에 조회할 수 있다.

## 한계

### 별칭 사용 불가

```sql
select t
From Team t
         -- as 사용 불가
         join fetch t.members as m
```

- fetch join 대상에는 별칭을 줄 수 없다.
    - 하이버네이트는 가능하지만 가급적 사용하지 않는다.

```sql
select t
From Team t
         -- 별칭으로 다시 걸러서 사용하는 방식 불가
         join fetch t.members as m
where m.username...
```

- fetch join은 연관된 모든 것을 가져오는 용도다.
    - 중간에 걸러서 가져오고 싶다면 fetch join을 쓰면 안된다.
    - 별칭을 줘서 선별적으로 가져왔다가 잘못하면 데이터가 누락되고 이상하게 돌아갈 수 있다.
- 원하는 회원만 가져오고 싶다면 처음부터 팀이 아니라 회원에 대한 쿼리를 날려야 한다.
- JPA는 객체 그래프 탐색으로 연관된 데이터를 모두 가져올 수 있다는 걸 가정한다.
    - 잘못된 설정으로 몇 개가 걸러져서 나온다면 이 상황을 보장할 수 없다.
- 유일하게 쓰는 케이스는 별칭 안에서 또 join fetch를 해서 들어가야할 때 뿐이다.

### 둘 이상의 컬렉션 불가

- 둘 이상의 컬렉션은 fetch join 할 수 없다.
    - 곱하기에 곱하기가 되면서 데이터가 기하급수적으로 늘어나기 때문이다.
    - ex. team이 members와 orders를 가진다면 이 둘을 한번에 fetch join할 수 없다.

### 페이징 API 사용 불가

- 컬렉션을 fetch join 하면 페이징 API를 사용할 수 없다.
- page size = 1이면 팀 A의 회원 2명이 다 나오는 게 아니라 짤려서 하나만 가져온다.
    - JPA는 팀 A의 결과가 2명임에도 1명밖에 없다고 말하게 된다.
    - 회원 2는 2 페이지에 있기 때문에 모른다.
- 일대일, 다대일 같은 단일 값 연관 필드는 fetch join으로도 페이징이 가능하다.
    - 데이터 뻥튀기가 되지 않기 때문이다.
- 하이버네이트는 경고 로그를 남기고 메모리에서 페이징하지만 매우 위험하다.
    - 데이터가 백만 건이면 다 일단 가져와서 그 안에서 페이징 하기 때문에 장애가 발생한다.

```sql
-- before
select t
From Team t
         join fetch t.members m
-- after
select m
From Member m
         join fetch m.team t
```

- 페이징을 하고 싶다면 쿼리를 바꿔야 한다.
- 쿼리를 반대로 뒤집으면 회원과 팀이 다대일이 되어 페이징이 가능해진다.

```java
public class JpaMain {

    public static void main(String[] args) {
        String query = "select t From t";

        List<Team> result = em.createQuery(query, Team.class)
                .setFirstResult(0)
                .setMaxResult(2)
                .getResultList();

        for (Team team : result) {
      ...
        }
    }
}
```

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%203.35.57.png)

- 쿼리를 수정할 수 없다면 maxResult를 건다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%203.37.48.png)

- 하지만 지연 로딩으로 팀 A, B의 회원을 2번 더 쿼리한다.
    - 즉, 쿼리가 총 3번이 나간다.
    - 그래서 fetch join으로 한 번에 불러오는 게 좋은데 페이징이 힘들다.

### @BatchSize

```java

@Entity
public class Team {
  
  ...

    @BatchSize(size = 100)
    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();
  
  ...
}
```

- 페이징 때문에 fetch join을 사용하지 않는다면 @BatchSize를 사용한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%203.42.55.png)

- 팀을 가져올 때 지연 로딩으로 되어있는 회원에 대해 `where team_id in (A, B)`로 가져온다.
- 지연 로딩은 N + 1이 발생하기 때문에 fetch join을 사용하지만 연관 관계가 컬렉션인 경우는 @BatchSize를 사용할 수 있다.
    - 회원 수만큼이 아니라 딱 팀에 맞춰서 최적화된 쿼리로 불러올 수 있다.
- 보통 1000 이하의 값으로 주면 된다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%203.47.14.png)

- batchSize 설정은 글로벌로 두고 쓸 수도 있다.

## fetch join의 특징

- 연관된 Entity들을 SQL 한 번으로 조회하므로 성능 최적화가 된다.
- Entity에 직접 적용하는 글로벌 로딩 전략보다 우선한다.
    - 글로벌 로딩 전략이 `@OneTonMany(fetch = FetchType.LAZY)` 지연 로딩이더라도 적용된다.
- 실무에서 글로벌 로딩 전략은 모두 지연로딩이다.
- 최적화가 필요한 곳은 fetch join을 적용한다.

## 한계

- 모든 것을 fetch join으로 해결할 수는 없다.
- 객체 그래프를 유지할 때 사용하면 효과적이다.
    - team.getMembers()
- 여러 테이블을 join해서 Entity가 가진 모양과 다른 결과를 내야 한다면?
    - 일반 join으로 필요한 데이터만 조회해서 DTO로 반환한다.