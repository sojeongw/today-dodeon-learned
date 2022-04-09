# 즉시 로딩과 지연 로딩

![](../../.gitbook/assets/kimyounghan-orm-jpa/08/스크린샷%202020-07-08%20오후%203.09.37.png)

- 단순히 Member 정보만 사용하는 로직이라면 Team까지 불러오는 것은 손해다.
- 이런 문제로 JPA는 지연 로딩을 지원한다.

## 지연 로딩

```java

@Entity
public class Member {
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn
    private Team team;
}
```

- `fetch = FetchType.LAZY`
    - 프록시 객체로 조회한다.
    - 즉, Member 클래스만 DB에서 조회하고 Team은 조회하지 않는다.

```java
public class App {
    public void printUserAndTeam(String memberId) {
        Team team = new Team();
        team.setName("teamA");
        em.persist(team);

        Member member1 = new Member();
        member1.setUsername("member1");
        member1.setTeam(team);
        em.persist(member1);

        em.flush();
        em.clear();

        // 이때는 프록시로 남아있다가
        Member m = em.find(Member.class, member1.getId());
        System.out.println("m = " + m.getTeam().getClass());

        System.out.println("=============");
        // 이 시점이 되어서야 필요한 정보를 얻기 위한 쿼리가 날아간다.
        // 지연 로딩을 사용하면 이렇게 연관된 데이터를 프록시로 가져온다.
        // m.getTeam()은 프록시로 가져오기 때문에 이 시점에는 Entity로 초기화되지 않는다.
        // getTeam().getName()으로 그 안에 있는 데이터를 조회해야할 때 쿼리가 나가면서 초기화 된다.
        m.getTeam().getName();
        System.out.println("=============");

        tx.commit();
    }
}
```

![](../../.gitbook/assets/kimyounghan-orm-jpa/08/스크린샷%202020-07-09%20오전%202.23.33.png)

- Member 조회 쿼리만 실행되었다.
- Team에 대한 내용은 프록시로 만들어져 있다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/08/스크린샷%202020-07-09%20오전%202.27.34.png)

- Team을 실제로 조회하는 시점이 되어서야 해당 쿼리를 보낸다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/08/스크린샷%202020-07-09%20오전%202.29.49.png)

- Member를 조회하면 당장 조회하지 않는 Team은 프록시 객체로 박아놓는다.
- member.getTeam().getName()으로 실제 사용하는 시점이 되면 DB를 조회해서 초기화 된다.
    - getTeam()이 아니라 그 안의 getName()으로 데이터를 실제 조회할 때임을 주의하자.

## 즉시 로딩

만약 Member와 Team 둘 다 자주 사용한다면 매번 쿼리가 두 번씩 나가게 되면서 성능상 손해를 보게 된다. 그래서 사용하는 것이 EAGER 로딩이다.

```java

@Entity
public class Member {
    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn
    private Team team;
}
```

```java
public class App {
    public void printUserAndTeam(String memberId) {
        Team team = new Team();
        team.setName("teamA");
        em.persist(team);

        Member member1 = new Member();
        member1.setUsername("member1");
        member1.setTeam(team);
        em.persist(member1);

        em.flush();
        em.clear();

        Member m = em.find(Member.class, member1.getId());
        System.out.println("m = " + m.getTeam().getClass());

        System.out.println("=============");
        m.getTeam().getName();
        System.out.println("=============");

        tx.commit();
    }
}
```

![](../../.gitbook/assets/kimyounghan-orm-jpa/08/스크린샷%202020-07-09%20오전%202.34.31.png)

- Member와 Team에 대해 쿼리가 한 번에 나간다.
- 따라서 Team에 대한 정보를 조회할 때 `getClass()`를 하면 프록시가 아니라 진짜 정보인 Team이 나온다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/08/스크린샷%202020-07-09%20오전%202.37.29.png)

즉시 로딩을 할 때는 두 가지 선택을 할 수 있다.

1. Member를 가지고 올 때 EAGER로 걸린 Entity를 join해서 한 방 쿼리로 가지고 온다.
    - 웬만한 JPA 쿼리는 이렇게 한 방에 가져오려고 한다.
2. 일단 Member를 가지고 온 다음 EAGER로 되어있는 Entity를 확인해서 한 번 더 쿼리를 날린다.
    - 즉, Member, Team에 em.find()를 각각 해서 두 번의 쿼리를 보낸다.
    - 두 번에 나눠서 하기 때문에 성능이 좋지 않다.

## 즉시 로딩 주의 사항

- 실무에서는 가급적 지연 로딩만 사용해야 한다.

### 예상치 못한 SQL 발생

- 즉시 로딩을 적용하면 예상하지 못한 SQL이 발생할 수 있다.
    - join이 한 두개일 때는 문제가 없다.
    - 데이터가 많아지면 join 때문에 성능에 부담이 된다.
        - 데이터가 10개면 `find()`할때 10개를 모두 join하면서 생각하지도 못한 거대 쿼리가 나간다.

### N+1 문제

- 즉시 로딩은 JPQL에서 N+1 문제를 일으킨다.
- 1은 최초 쿼리, N은 최초 쿼리의 결과 개수다.
- 최초에 한 번 쿼리 했는데 그 결과 개수만큼 추가적인 쿼리가 나가는 것이다.

```java

@Entity
public class Member {
    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn
    private Team team;
}
```

```java
public class App {
    public void printUserAndTeam(String memberId) {
        Team team = new Team();
        team.setName("teamA");
        em.persist(team);

        Member member1 = new Member();
        member1.setUsername("member1");
        member1.setTeam(team);
        em.persist(member1);

        em.flush();
        em.clear();

        // em.find() 대신 JPQL을 사용하는 경우
        // member만 가지고 온다.
        List<Member> members = em.createQuery("select m from Member m", Member.class).getResultList();

        // member만 가지고 와보니 team에 즉시 로딩이 걸려있는 걸 확인한다.
        // 그럼 member 쿼리 한 번만으로 끝나는 게 아니라
        // select * from Team where team_id = ?로 관련된 team을 조회하는 쿼리가 같이 나간다.

        tx.commit();
    }
}
```

![](../../.gitbook/assets/kimyounghan-orm-jpa/08/스크린샷%202020-07-09%20오전%202.52.01.png)

- EAGER로 했는데도 불구하고 쿼리를 두 번 날리고 있다.
- `find()`는 PK를 찍어서 가져오는 것이기 때문에 JPA가 내부적으로 최적화를 할 수 있다.
- 하지만 JPQL은 코드에 적은 쿼리가 그대로 날아가기 때문에 정직하게 Member만 조회한다.
    - Member를 가지고 와봤더니 Team이 즉시 로딩으로 설정되어 있다.
    - 즉시 로딩은 가져올 때 무조건 값이 다 들어가있어야 한다.
    - 따라서 EAGER로 되어있는 Team을 조회하기 위해 10개의 쿼리가 다시 나간다.
    - Member가 10개라면, Member 조회 쿼리 한 번 나가고, 각 10개 Member의 Team을 다시 쿼리한다.

```java
public class App {
    public void printUserAndTeam(String memberId) {
        Team teamA = new Team();
        teamA.setName("teamA");
        em.persist(teamA);

        Team teamB = new Team();
        teamB.setName("teamB");
        em.persist(teamB);

        Member member1 = new Member();
        member1.setUsername("member1");
        member1.setTeam(teamA);
        em.persist(member1);

        Member member2 = new Member();
        member2.setUsername("member2");
        member2.setTeam(teamB);
        em.persist(member2);

        em.flush();
        em.clear();

        // 일단 멤버를 가지고 왔더니 멤버가 2개가 있고 각각 다른 팀을 가지고 있다.
        // 다른 팀이면 영속성 컨텍스트에서 다시 가져올 수도 없다.
        // 다 따로따로 가져와야 하기 때문에 각 팀을 가져올 때마다 쿼리가 한 번씩 추가로 나간다.
        List<Member> members = em.createQuery("select m from Member m", Member.class).getResultList();

        tx.commit();
    }
}
```

![](../../.gitbook/assets/kimyounghan-orm-jpa/08/스크린샷%202020-07-09%20오전%203.03.03.png)

- 일단 Member를 가지고 온다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/08/스크린샷%202020-07-09%20오전%203.03.50.png)

- 그리고 멤버 2명이 가지고 있는 Team에 대해 다시 쿼리한다.
- 최초 쿼리를 한 번 날렸는데 그것 때문에 추가 쿼리를 N번 날려야 한다.

```java

@Entity
public class Member {
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn
    prviate Team
    team;
}
```

![](../../.gitbook/assets/kimyounghan-orm-jpa/08/스크린샷%202020-07-09%20오전%203.06.38.png)

- 지연 로딩으로 설정한다면 당장 Team을 쓰지 않아 프록시로 박혀있기 때문에 쿼리 하나로 끝난다.

## N+1 해결 방법

### 지연 로딩

- 기본적으로 모든 연관 관계를 lazy로 설정한다.
- `@ManyToOne`, `@OneToOne`
    - 기본 값이 즉시 로딩이므로 지연 로딩으로 설정한다.
- `@OneToMany`, `@ManyToMany`
    - 기본 값이 지연 로딩이다.

### fetch join

- 런타임에 동적으로 내가 원하는 정보만 선택해서 가져오는 방법
- join을 이용해 쿼리를 한 번만 날린다.
    - 상황에 따라 Member만 필요한 곳은 Member만, Team까지 필요하면 둘 다 불러온다.

```java
public class App {
    public void printUserAndTeam(String memberId) {
        Team teamA = new Team();
        teamA.setName("teamA");
        em.persist(teamA);

        Team teamB = new Team();
        teamB.setName("teamB");
        em.persist(teamB);

        Member member1 = new Member();
        member1.setUsername("member1");
        member1.setTeam(teamA);
        em.persist(member1);

        Member member2 = new Member();
        member2.setUsername("member2");
        member2.setTeam(teamB);
        em.persist(member2);

        em.flush();
        em.clear();

        // fetch join으로 한 방 쿼리를 날린다.
        List<Member> members = em.createQuery("select m from Member m join fetch m.team", Member.class).getResultList();

        // 이제 다 값이 채워져서 어떤 값을 조회해도 쿼리가 나가지 않는다.

        tx.commit();
    }
}
```

![](../../.gitbook/assets/kimyounghan-orm-jpa/08/스크린샷%202020-07-09%20오전%203.13.12.png)

- 지연 로딩을 사용하더라도 Team 데이터를 조회하게 되면 쿼리는 계속 나간다.
- fetch join을 하면 한 번에 Team 정보까지 모두 가져오게 된다.
    - 이미 한 방 쿼리로 값이 채워져 있기 때문에 순회하면서 데이터를 조회해도 전혀 문제가 없다.

이 밖에도 애너테이션이나 배치 사이즈로 푸는 방법이 있지만 기본적으로 지연 로딩과 fetch join을 사용한다.

### 지연 로딩 활용

- 예제는 즉시 로딩도 섞었지만 **실무에서는 모두 지연 로딩**으로 해야 한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/08/screenshot%202021-03-20%20오후%2010.29.54.png)

- 회원과 팀, 주문과 상품은 자주 묶어서 조회한다. 
- 회원과 주문은 가끔만 사용한다.
- 그럼 전자는 즉시 로딩, 후자는 지연 로딩으로 하면 된다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/08/screenshot%202021-03-20%20오후%2010.29.58.png)

- 회원을 조회하면 팀은 즉시 로딩이므로 쿼리 한 번으로 조회 한다.
- 주문 내역은 지연 로딩이므로 직접 사용할 때 가져온다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/08/screenshot%202021-03-20%20오후%2010.30.02.png)

- 회원 조회 시에 주문 내역 프록시를 한 번 조회했다면 주문 내역에 연관된 상품도 즉시 로딩으로 한 번에 가져온다.

### 정리

- 모든 연관 관계에 지연 로딩을 사용하자.
- 실무에서 즉시 로딩을 사용하지 말자.
- JPQL fetch 조인이나 Entity 그래프 기능을 사용하자.
- 즉시 로딩은 상상하지 못한 쿼리가 나간다.
