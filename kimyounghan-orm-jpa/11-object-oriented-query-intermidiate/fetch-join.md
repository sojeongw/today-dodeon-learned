# fetch join

- SQL 조인 종류는 아니다.
- JPQL에서 성능 최적화를 위해 제공하는 기능이다.
- 연관된 엔티티나 컬렉션을 SQL 한 번에 조회할 수 있다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%202.27.05.png)

- `select m`만 썼는데도 회원을 조회하면서 연관된 팀도 SQL 한 번에 조회한다.
- SQL을 보면 회원 `M.*`과 팀 `T.*`을 함께 select 한다.

즉시 로딩으로 가져오는 방법과 똑같다. 단지 join fetch라고 명시적으로 선언해서 원하는 객체 그래프를 한 번에 조회하는 것이다.

## 단일 값 fetch join

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%202.31.43.png)

회원과 팀이 이런식으로 구성되어 있다고 해보자.

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%202.31.53.png)

fetch join을 통해 회원과 팀을 한 번에 가져온다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%202.31.49.png)

그럼 이렇게 한 번에 결과를 조회할 수 있다. 내부 조인이기 때문에 팀이 없는 회원은 누락되었다.

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

위의 코드는 연관 관계에 있는 team이 프록시로 들어왔다가 실제 사용할 때 지연 로딩으로 쿼리가 나간다.

회원 1이 팀 A를 불러 올 때 최초 SQL을 날리고 회원 2는 같은 팀 A이므로 1차 캐시에서 가져온다. 회원 3은 팀 B이기 때문에 영속성 컨텍스트에 존재하지 않으므로 새로운
SQL을 날린다.

소속이 모두 다르면 팀마다 한 번씩 쿼리가 다 나가야 하므로 비효율적이다. 흔히 N+1이라고 부르는 이 문제는, 회원을 가져오기 위해 최초에 날린 1번의 쿼리에, 그 결과만큼 즉
팀 개수만큼 N번을 날린다.

```java
public class JpaMain {

  public static void main(String[] args) {
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

이 코드는 fetch join으로 회원과 팀을 함께 조회해서 지연 로딩되지 않는다. 즉, for문 안의 team은 프록시가 아니라 실제 엔티티가 담겨있다.

지연 로딩으로 설정이 되어있어도 fetch join이 우선 순위를 가진다.

## 컬렉션 fetch join

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%202.47.42.png)

팀 입장에서 멤버를 반대로 조인하는 상황이다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%202.50.49.png)

팀 A가 중복으로 출력되었다. DB 입장에서는 일대다 join 시 데이터가 뻥튀기 되기 때문이다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%202.48.36.png)

팀 A 입장에서는 조인을 하면

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%202.48.40.png)

회원이 2명이므로 데이터를 위와 같이 만들어낸다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%202.48.45.png)

따라서 결과가 2줄이 되는 것이다. 이 데이터는 row는 2개이지만 영속성 컨텍스트에 있는 ID 1번 값을 같이 공유해 쓰고 있는 것이기 때문에 주소 값이 같다.

단순히 `select t From Team t`만 하면 결과는 2개가 나온다. 하지만 `fetch join`이 들어가면서 중복 결과까지 합한 3개가 나오는 것이다. 다대일과는
다르게 일대다는 이런 조인에 의한 뻥튀기를 조심해야 한다.

JPA 입장에서는 회원이 몇 명 있는지 알 수가 없어서 DB가 가져오는 대로 가져와야 한다. 객체 입장에서 몇 명인지 미리 알고 대처할 수 있는 일이 아니다.

## fetch join과 DISTINCT

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%203.03.07.png)

- SQL의 DISTINCT
    - 중복을 완벽하게 제거할 수 없다.

SQL에만 DISTINCT를 추가하면 PK는 다르기 때문에 데이터가 다르므로 중복 제거에 실패한다. 100% 똑같아야만 DISTINCT가 먹는다.

```sql
select distinct t
from Team t
         join fetch t.members
where t.name = '팀A'
```

- JPQL의 DISTINCT
    - SQL에 DISTINCT를 추가한다.
    - SQL 결과물이 애플리케이션에 올라오면 엔티티 중복을 제거한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%203.05.25.png)

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%203.06.47.png)

JPQL의 DISTINCT는 애플리케이션에서 추가적으로 중복 제거를 시도한다. 같은 식별자를 가진 Team 엔티티를 제거하는 것이다.

## 일반 join과의 차이

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%203.09.15.png)

JPQL은 결과를 반환할 때 연관 관계를 고려하지 않는다. 단지 select 절에 지정한 엔티티만 조회할 뿐이다. 예제에서는 Team 엔티티만 조회하고 Member 엔티티는
조회하지 않는다.

fetch join을 사용할 때만 연관된 엔티티를 함께 조회한다. 즉시 로딩의 개념인 것이다. 객체 그래프를 SQL 한 번에 조회할 수 있다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%203.11.03.png)

일반 join은 실행할 때 연관된 엔티티를 함께 조회하지 않는다. 실제 데이터는 team만 가져온다. 다만 예제의 for문에서 team.getMembers()를 할 때 쿼리가 다시
나가면서 조회되는 것이다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%203.13.12.png)

반면 join fetch는 한 번에 member 정보까지 가져오는 것을 볼 수 있다. join fetch로 대부분의 N+1 문제를 해결할 수 있다.

## 한계

### 별칭 사용 불가

```sql
-- as 사용 불가
select t
From Team t
         join fetch t.members as m
```

- fetch join 대상에는 별칭을 줄 수 없다.
- 하이버네이트는 가능하지만 가급적 사용하지 않는다.

fetch join은 연관된 모든 것을 가져오는 것이다. 중간에 몇 개를 걸러서 가져오고 싶다면 fetch join을 쓰면 안된다. 별칭을 줘서 선별적으로 가져왔다가 잘못 조작하게
되면 데이터가 누락되고 이상하게 돌아갈 수 있다.

객체 그래프에서 점을 찍어 나갈 때 원래 5명이 나가야하는데 `where m.age > 10`으로 3명만 나오면 JPA가 의도한 설계와 맞지 않게 된다. fetch join은
무조건 데이터를 다 구해야 한다.

만약 원하는 데이터만 가져오고 싶다면 처음부터 팀이 아니라 회원에 대한 쿼리를 날리면 된다. 객체 그래프를 탐색한다는 것은 연관된 데이터를 모두 가져온다는 걸 의미하는 것이기
때문이다.

유일하게 쓰는 케이스는 별칭 안에서 또 join fetch를 해서 들어가야할 때 뿐이다.

### 둘 이상의 컬렉션 불가

- 둘 이상의 컬렉션은 fetch join 할 수 없다.

team이 members와 orders를 가진다면 이 둘을 한번에 fetch join할 수 없다. 곱하기에 곱하기가 되면서 데이터가 기하급수적으로 늘어나기 때문이다.

### 페이징 API 사용 불가

- 컬렉션을 fetch join 하면 페이징 API를 사용할 수 없다.

일대일, 다대일 같은 단일 값 연관 필드는 fetch join으로도 페이징이 가능하다. 데이터 뻥튀기가 되지 않기 때문이다.

하지만 반대의 상황은 데이터 뻥튀기가 된다. page size 1의 데이터를 가져와야 하면 팀 A에 대한 회원 2명이 다 나오는 게 아니라 짤리고 하나만 가져온다. JPA 입장에서
팀 A에는 결과가 두 개임에도 하나밖에 없다고 말하는 상황이 되어버린다. 회원 2는 2페이지에 있기 때문에 모르는 것이다.

하이버네이트는 경고 로그를 남기고 메모리에서 페이징하지만 매우 위험하다. 데이터가 백만 건이면 백만 건을 다 일단 가져와서 그 안에서 페이징 하기 때문에 장애나기 쉽다.

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

만약 페이징을 하고 싶다면 뭐리를 바꿔야 한다. 쿼리를 반대로 뒤집는다.

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

일반적으로 페이징을 하면 team을 조회할 때 limit을 잘 걸어서 가져온다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%203.37.48.png)

문제는 for문을 돌 때다. 각 팀마다 멤버를 지연 로딩으로 2번 불러온다. 즉 쿼리가 총 3번이 나간다. 그래서 fetch join으로 한 번에 불러오면 좋은데 페이징이 힘든 게
문제다.

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

fetch join을 쓰는 대신 `@BatchSize`를 쓸 수 있다.. 보통 1000 이하의 값으로 주면 된다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%203.42.55.png)

결과를 보면 team_id가 2개 들어가있다. 한 번에 팀 A, B에 대항되는 회원을 다 가져온 것이다. 지연 로딩으로 설정되어 있지만 끌고 올 때 in 쿼리로 100개씩 넘기기
때문이다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/11/screenshot%202021-05-23%20오후%203.47.14.png)

batchSize 설정은 글로벌로 두고 쓸 수도 있다.

## 특징

- 연관된 엔티티들을 SQL 한 번으로 조회하므로 성능 최적화가 가능하다.
- 엔티티에 직접 적용하는 글로벌 로딩 전략보다 우선한다.
    - 글로벌 로딩 전략: `@OneTonMany(fetch = FetchType.LAZY)`
- 실무에서 글로벌 로딩 전략은 모두 지연로딩이다.
- 최적화가 필요한 곳은 fetch join을 적용한다.

## 한계

- 모든 것을 fetch join으로 해결할 수는 없다.
- 객체 그래프를 유지할 때 사용하면 효과적이다.
- 여러 테이블을 join해서 엔티티가 가진 모양과 다른 결과를 내야 한다면?
    - 일반 join을 사용하고 필요한 데이터만 조회해서 DTO로 반환한다.