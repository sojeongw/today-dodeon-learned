# 연관 관계 매핑

## 목표

- 객체와 테이블 연관 관계의 차이를 이해한다.
    - 객체의 참조와 테이블의 외래 키를 매핑하는 방법

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

객체를 테이블에 맞춰 모델링 했을 경우 연관 관계가 없이 이러한 형태가 나온다.

{% endtab %} {% tab title="Entity 정의" %}

```java

@Entity
public class Member {

  private Long id;
  @Column(name = "USERNAME")
  private String name;

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
// 팀 저장
    Team team = new Team();
    team.setName("TeamA");
    em.persist(team);

// 회원 저장
    Member member = new Member();
    member.setName("member1");
    member.setTeamId(team.getId());
    em.persist(member);
  }
}
```

{% endtab %} {% tab title="Entity 조회" %}

```java
public class JpaMain() {

  public static void main(String[] args) {
    // 조회
    Member findMember = em.find(Member.class, member.getId());

    // 연관 관계가 없음
    Team findTeam = em.find(Team.class, team.getId());
  }
}
```

{% endtab %}{% endtabs %}
