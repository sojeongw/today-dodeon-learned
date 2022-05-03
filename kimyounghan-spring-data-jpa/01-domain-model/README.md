# 예제 도메인 모델

![](../../.gitbook/assets/kimyounghan-spring-data-jpa/01/screenshot%202022-05-07%20오후%203.10.00.png)

![](../../.gitbook/assets/kimyounghan-spring-data-jpa/01/screenshot%202022-05-07%20오후%203.10.15.png)

{% tabs %} {% tab title="Member.java" %}

```java

@Entity
@Getter
@Setter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
// 대상을 적는다. team을 적으면 무한 루프가 되므로 주의한다.
@ToString(of = {"id", "username", "age"})
public class Member {

    @Id
    @GeneratedValue
    @Column(name = "member_id")
    private Long id;

    private String username;
    private int age;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "team_id")  // FK 이름
    private Team team;

    public Member(String username) {
        this.username = username;
    }

    // 연관 관계 메서드로 반대쪽 멤버도 바꿔줘야 한다.
    public void changeTeam(Team team) {
        this.team = team;
        team.getMembers().add(this);
    }
}
```

{% endtab %} {% tab title="Team.java" %}

```java

@Entity
@Getter
@Setter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@ToString(of = {"id", "name"})
public class Team {

    @Id
    @GeneratedValue
    @Column(name = "team_id")
    private Long id;
    private String name;

    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();

    public Team(String name) {
        this.name = name;
    }
}
```

{% endtab %} {% endtabs %}

- @NoArgsConstructor(access = AccessLevel.PROTECTED)
    - JPA는 기본적으로 파라미터가 없는 기본 생성자가 필요하다.
    - protected까지만 허용된다.
- ToString
    - 순환 참조를 조심한다.
- 연관 관계 편의 메서드로 양쪽 값을 세팅한다.

```text
member = Member(id=5, username=member3, age=30)

select team0_.team_id as team_id1_1_0_, team0_.name as name2_1_0_ from team team0_ where team0_team_id=?
select team0_.team_id as team_id1_1_0_, team0_.name as name2_1_0_ from team team0_ where team0_.team_id=2;

member.team = Team(id=2, name=teamB)
```

- @ManyToOne
    - 기본이 EAGER이기 때문에 LAZY로 바꾼다.
- member.team을 가져올 때 team에 대한 쿼리가 다시 나간다. 