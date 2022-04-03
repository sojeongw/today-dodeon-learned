# 단방향 연관 관계

![](../../.gitbook/assets/kimyounghan-orm-jpa/05/screenshot%202021-03-19%20오후%209.58.29.png)

- 객체 지향적으로 모델링을 하면 Member에는 teamId가 아니라 Team 참조값을 그대로 갖게 된다.

{% tabs %} {% tab title="Entity 정의" %}

```java

@Entity
public class Member {

    private Long id;
    @Column(name = "USERNAME")
    private String name;

    // 이제 FK 대신 객체를 참조하기 위해 삭제한다.
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
        // id 대신 team 객체를 바로 넣어 주면 알아서 FK로 insert 한다.
        member.setTeam(team);

        em.persist(member);

        // 조회
        // member는 앞에서 이미 영속성 컨텍스트에 들어가 있어서
        // 1차 캐시에 저장이 되어있는 상태이기 때문에 조회 쿼리는 나가지 않는다.
        // 만약 쿼리를 직접 보고 싶다면
        // em.flush()   싱크를 맞추고
        // em.clear()   영속성 컨텍스트를 초기화 하면 findMember부터는 깔끔한 영속성 컨텍스트에서 다시 가져오면서 쿼리 로그를 볼 수 있다.
        Member findMember = em.find(Member.class, member.getId());

        // 이전에는 member.getTeamId()를 사용했는데 이제 바로 접근할 수 있다.
        Team findTeam = findMember.getTeam();

        tx.commit();
    }
}
```

{% endtab %}{% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/05/screenshot%202021-03-19%20오후%2010.09.33.png)

- FK 대신 객체를 참조하면서 연관 관계가 매핑되었다.

## 연관 관계 수정

{% endtab %} {% tab title="연관 관계 수정" %}

```java
public class JpaMain() {

    public static void main(String[] args) {
        
        ...

        // 만약 다른 팀으로 바꾸고 싶다면
        Team teamB = new Team();
        teamB.setName("TeamB");

        em.persist(teamB);

        // 그냥 다시 새로운 팀으로 넣어주면 된다.
        member.setTeam(teamB);

        // 그냥 값만 바꾸면 변경 감지해서 update 쿼리가 나간다.
    }
}
```

{% endtab %}{% endtabs %}

- 연관 관계를 수정할 때는 그냥 값만 바꿔 넣어주면 알아서 update 쿼리가 나간다.