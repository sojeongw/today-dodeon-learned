# 즉시 로딩과 지연 로딩

![](../.gitbook/assets/inflearn-orm-jpa/01/스크린샷%202020-07-08%20오후%203.09.37.png)

다시 이 질문으로 돌아가보자. 단순히 Member 정보만 사용하는 로직이라면 Team까지 불러오는 것은 손해다. 이런 문제로 JPA는 지연 로딩을 지원하고 있다.

## 지연 로딩

```java
@Entity
public class Member {
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn
    prviate Team team;
}
```

`fetch = FetchType.LAZY`로 설정하면 프록시 객체로 조회한다. 즉, Member 클래스만 DB에서 조회할 수 있다.

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
        // Team에 대한 정보는 이 시점이 되어서야 쿼리가 날아간다.
        m.getTeam().getName();
        System.out.println("=============");

        tx.commit();
    }
}
```

![](../.gitbook/assets/inflearn-orm-jpa/01/스크린샷%202020-07-09%20오전%202.23.33.png)

당장 조회하려는 Member에 대한 쿼리만 실행되었다. 그리고 Team에 대한 내용은 프록시로 만들어져 있다.

![](../.gitbook/assets/inflearn-orm-jpa/01/스크린샷%202020-07-09%20오전%202.27.34.png)

따라서 Team을 실제로 조회하는 시점이 되어서야 해당 쿼리를 보낸다.

![](../.gitbook/assets/inflearn-orm-jpa/01/스크린샷%202020-07-09%20오전%202.29.49.png)

정리하자면 이렇다. Member를 조회하면 당장 조회하지 않는 Team은 프록시 객체로 박아놓는다.

그리고 `member.getTeam()`으로 실제 사용하는 시점이 되면 DB를 조회해서 초기화가 된다. 즉, 쿼리가 날아간다.

## 즉시 로딩

만약 Member와 Team 둘 다 자주 사용한다면 매번 쿼리가 두 번씩 나가게 되면서 성능상 손해를 보게 된다. 그래서 사용하는 것이 EAGER 로딩이다.

```java
@Entity
public class Member {
    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn
    prviate Team team;
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

즉시 로딩으로 바꾸고 다시 코드를 실행하면, 

![](../.gitbook/assets/inflearn-orm-jpa/01/스크린샷%202020-07-09%20오전%202.34.31.png)

Member와 Team에 대해 한 번에 쿼리가 나간다. 따라서 Team에 대한 정보를 조회할 때 `getClass()`를 하면 프록시가 아니라 진짜 정보인 Team이 나온다.

![](../.gitbook/assets/inflearn-orm-jpa/01/스크린샷%202020-07-09%20오전%202.37.29.png)

정리하면 위와 같다. 

즉시 로딩을 할 때는 두 가지 선택을 할 수 있다. 

1. Member를 가지고 올 때 EAGER로 걸린 엔티티를 join해서 한 방 쿼리로 가지고 온다.
    - 웬만한 JPA 쿼리는 이렇게 한 방에 가져오려고 한다.
2. 일단 Member를 가지고 온 다음 EAGER로 되어있는 엔티티를 확인해서 한 번 더 쿼리를 날린다. 즉, Member 조회 + Team 조회로 두 번의 쿼리를 보낸다.
    - 성능이 좋지 않다.
    
## 즉시 로딩 주의 사항

특히 실무에서는 가급적 지연 로딩만 사용해야 한다. 

### 예상치 못한 SQL 발생

즉시 로딩을 적용하면 예상하지 못한 SQL이 발생할 수 있다. 데이터가 많아지면 join 때문에 성능에 부담이 된다. 데이터가 10개면 `find()`할때 10개를 모두 join해야 한다. 생각하지도 못한 거대한 쿼리가 나가게 되는 것이다.

### N+1 문제

즉시 로딩은 JPQL에서 N+1 문제를 일으킨다.

```java
@Entity
public class Member {
    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn
    prviate Team team;
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

        List<Member> members = em.createQuery("select m from Member m", Member.class).getResultList();

        tx.commit();
    }
}
```

![](../.gitbook/assets/inflearn-orm-jpa/01/스크린샷%202020-07-09%20오전%202.52.01.png)

EAGER로 했는데도 불구하고 쿼리를 두 번 날리고 있다. `find()`는 PK를 찍어서 가져오는 것이기 때문에 JPA가 내부적으로 최적화를 할 수 있다. 하지만 JPQL은 코드에 적은 쿼리가 그대로 날아간다. 즉, 쓰여있는 Member만 조회한다. 

Member를 가지고 와봤더니 Team이 즉시 로딩으로 설정되어 있고 즉시 로딩은 가져올 때 무조건 값이 다 들어가있어야 하므로, Member 쿼리 하나 나간 것에 더해 Member가 10개면 EAGER로 되어있는 데이터를 조회하기 위해 10개의 쿼리가 다시 나가는 것이다.

LAZY면 그냥 프록시로 넣으면 되겠지만 EAGER면 반환 시점에 이미 값이 들어가있어야 하기 때문에 부랴부랴 다시 쿼리를 치는 것이다.

SQL로 보자면 `select * from Member` 후에 `select * from Team where TEAM_ID = ?`로 다시 나간다고 할 수 있다.

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

        List<Member> members = em.createQuery("select m from Member m", Member.class).getResultList();

        tx.commit();
    }
}
```

각자 다른 Team을 가진 멤버를 조회해보자.

![](../.gitbook/assets/inflearn-orm-jpa/01/스크린샷%202020-07-09%20오전%203.03.03.png)

일단 Member를 한 번에 가지고 온다.

![](../.gitbook/assets/inflearn-orm-jpa/01/스크린샷%202020-07-09%20오전%203.03.50.png)

그리고 Team A, B에 대한 쿼리를 각각 보낸다. 최초 쿼리를 한 번 날렸는데 그것 때문에 추가 쿼리를 N번 날려야 하는 상황인 것이다.

```java
@Entity
public class Member {
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn
    prviate Team team;
}
```

하지만 LAZY로 설정한다면?

![](../.gitbook/assets/inflearn-orm-jpa/01/스크린샷%202020-07-09%20오전%203.06.38.png)

당장 Team을 쓰지 않아 프록시로 박혀있기 때문에 쿼리 하나로 처리가 된다. 

## N+1 해결 방법

### 지연 로딩

`@ManyToOne`, `@OneToOne`은 기본이 즉시 로딩이므로 지연 로딩으로 설정한다. `@OneToMany`, `@ManyToMany`는 기본이 지연 로딩이다.    
    
### fetch join

런타임에 동적으로 내가 원하는 정보만 선택해서 가져오는 방법이다. 예를 들어 화면에 따라 Member만 필요한 곳은 Member만, Team까지 필요하면 둘 다 불러오도록 하는 것이다. join으로 한 방 쿼리를 날려 가져오게 된다.

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

        // fetch join
        List<Member> members = em.createQuery("select m from Member m join fetch m.team", Member.class).getResultList();

        tx.commit();
    }
}
```

![](../.gitbook/assets/inflearn-orm-jpa/01/스크린샷%202020-07-09%20오전%203.13.12.png)

지연 로딩을 사용하더라도 Team을 루프로 돌리면 쿼리는 계속 나간다. 하지만 fetch join을 하면 한 번에 Team 정보까지 모두 가져오게 된다. 이미 값이 채워져 있기 때문에 루프를 돌려서 조회해도 전혀 문제가 없다.