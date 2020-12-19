# 단방향 연관 관계

![](../../.gitbook/assets/kimyounghan-orm-jpa/05/screenshot%202021-03-19%20오후%209.58.29.png)

객체 지향적으로 모델링을 하면 Member에는 teamId가 아니라 Team 참조값을 그대로 갖게 된다.

{% tabs %} {% tab title="Entity 정의" %}

```java

@Entity
public class Member {

    private Long id;
    @Column(name = "USERNAME")
    private String name;

//  @Column(name = "TEAM_ID")
//  private Long teamId;

    // 하나의 팀에 여러 멤버가 소속될 수 있으므로
    // Member 입장에서는 many, Team 입장에서는 one
    @ManyToOne
    // join 하기 위한 FK를 명시해준다.
    @JoinColumn(name = "TEAM_ID")
    private Team team;
}

@Entity
public class Team {

    @Id
    @GeneratedValue
    private Long id;

    private String name;
}
```

{% endtab %} {% tab title="Entity 저장 및 조회" %}

```java
public class JpaMain() {

    public static void main(String[] args) {
        // 팀 저장
        Team team = new Team();
        team.setName("TeamA");
        em.persist(team);

        // 회원 저장
        Member member = new Member();
        member.setName("member1");

        // 팀을 바로 넣어주면 알아서 FK로 insert 한다.
        member.setTeam(team);
        em.persist(member);

        // 조회
        // 1차 캐시에 저장이 되어있는 상태이기 때문에 조회 쿼리는 나가지 않는다.
        // 만약 쿼리를 직접 보고 싶다면
        // em.flush()로 영속성 컨텍스트에 있는 쿼리를 날린 다음
        // em.clear()로 영속성 컨텍스트를 완전 초기화 시킨다.
        Member findMember = em.find(Member.class, member.getId());

        // 멤버에서 바로 접근할 수 있다.
        Team findTeam = findMember.getTeam();
        
        tx.commit();
    }
}
```

{% endtab %}{% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/05/screenshot%202021-03-19%20오후%2010.09.33.png)

## 연관 관계 수정

{% endtab %} {% tab title="연관 관계 수정" %}

```java
public class JpaMain() {

    public static void main(String[] args) {
        Team teamB = new Team();
        teamB.setName("TeamB");
        em.persist(teamB);
        
        // 새로운 팀 저장
        member.setTeam(teamB);
    }
}
```

{% endtab %}{% endtabs %}

