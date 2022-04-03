# 연관 관계 매핑

## 목표

- 객체와 테이블 연관 관계의 차이를 이해한다.
    - 객체는 그래프로 쭉쭉 참조할 수 있지만 테이블은 외래키를 가지고 접근한다.
- 객체의 참조와 테이블의 외래 키를 매핑하는 방법을 알아본다.

## 용어 이해

- 방향(Direction)
    - 단방향, 양방향
- 다중성(Multiplicity)
    - 다대일, 일대다, 일대일, 다대다
- 연관 관계의 주인(Owner)
    - 객체 양ㅂ아향 연관 관계는 관리하는 주인이 필요하다.

## 연관 관계가 필요한 이유

> 객체 지향 설계의 목표는 자율적인 객체들의 협력 공동체를 만드는 것이다.

객체 지향적으로 설계하기 위해서는 이해가 필요하다.

## 예제 시나리오

- 회원과 팀이 있다.
- 회원은 하나의 팀에만 소속될 수 있다.
- 회원과 팀은 다대일 관계다.
    - 하나의 팀에 여러 회원이 소속될 수 있다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/05/Screen%20Shot%202021-03-17%20at%2012.47.16%20PM.png)

- 객체 연관 관계
    - 테이블 내용에 맞춰서 모델링하면 teamId를 member가 그대로 가져간다.
    - team을 레퍼런스로 가져가야 하는데 DB처럼 teamId만 가지고 있다.
    - 연관 관계가 없는 형태가 나온다.
- 테이블 연관 관계
    - member에 team_id가 있다는 건 각 member가 어느 팀에 소속 됐는지 표시하는 것이다.
    - 즉, team은 여러 member를 가질 수 있다는 뜻이다.

{% tabs %} {% tab title="Entity 정의" %}

```java

@Entity
public class Member {

    private Long id;
    
    @Column(name = "USERNAME")
    private String name;

    // 테이블처럼 FK만 가지고 있는다.
    @Column(name = "TEAM_ID")
    private Long teamId;
}

@Entity
public class Team {

    @Id
    @GeneratedValue
    private Long id;

    private String name;
}
```

{% endtab %} {% tab title="Entity 저장" %}

```java
public class JpaMain() {

    public static void main(String[] args) {
        
        ...
      
        // 팀 저장
        Team team = new Team();
        team.setName("TeamA");
        
        // 영속 상태엔 일단 insert 해서 PK를 가지고 있다.
        em.persist(team);

        // 회원 저장
        Member member = new Member();
        member.setName("member1");
        // 앞서 영속화로 가지고 있던 teamId를 이용해 회원을 팀에 소속시킨다.
        member.setTeamId(team.getId());
        
        em.persist(member);
    }
}
```

{% endtab %} {% tab title="Entity 조회" %}

```java
public class JpaMain() {

    public static void main(String[] args) {
        
        ...
      
        // 조회
        // 회원은 teamId만 가지고 있다.
        Member findMember = em.find(Member.class, member.getId());

        // teamId만 가지고 있으니 이렇게 그래프로 한 번에 불러올 수가 없다. 
        // Team findTeam = em.find(Team.class, team.getId());
      
        // 그래서 teamId로
        Long findTeamId = findMember.getTeamId();
        // 다시 team을 찾아야 하는 번거로움이 생긴다.
        Team findTeam = em.find(Team.class, findTeamId);
        
    }
}
```

{% endtab %}{% endtabs %}

- 테이블은 외래키로 조인해서 연관된 테이블을 찾는다.
- 객체는 참조를 사용해 연관된 객체를 찾는다.
- 객체를 테이블에 맞춰 데이터 중심으로 모델링하면, 협력 관계를 만들기 힘들다.

