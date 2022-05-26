# 예제 도메인 모델

## 엔티티

![](../../.gitbook/assets/kimyounghan-querydsl/02/screenshot%202022-06-04%20오후%204.17.40.png)

## ERD

![](../../.gitbook/assets/kimyounghan-querydsl/02/screenshot%202022-06-04%20오후%204.17.54.png)

## 예제 코드

{% tabs %} {% tab title="Member.java" %}

```java

@Entity
@Getter
@Setter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@ToString(of = {"id", "username", "age"})
public class Member {

    @Id
    @GeneratedValue
    @Column(name = "member_id")
    private Long id;

    private String username;

    private int age;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "team_id")
    private Team team;

    public Member(String username) {
        this(username, 0);
    }

    public Member(String username, int age) {
        this(username, age, null);
    }

    public Member(String username, int age, Team team) {
        this.username = username;
        this.age = age;
        if (team != null) {
            changeTeam(team);
        }
    }

    // 연관 관계 편의 메서드
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

- Member.team이 연관 관계의 주인이다.
    - Member.team이 DB FK 값을 변경할 수 있다.
    - Team.members는 읽기만 가능하다.