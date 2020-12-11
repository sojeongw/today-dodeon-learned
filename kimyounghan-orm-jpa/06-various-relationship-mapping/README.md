# 다양한 연관 관계 매핑

## 연관 관계 매핑 시 고려사항 3가지

### 다중성

- @ManyToOne
- @OneToMany
- @OneToOne
- @ManyToMany

JPA 애너테이션은 DB와 매핑하기 위해 쓰는 것이기 때문에, 데이터베이스 관점에서의 다중성을 고민하면 된다.

다중성이 애매할 때는 반대쪽을 생각하면 된다. 회원과 팀이면 반대로 팀과 회원의 관계를 생각해보면 된다. 일대일의 반대는 일대일, 일대다의 반대는 다대일, 다대다의 반대는 다대다로
대칭성이 있기 때문이다.

참고로 사실 다대다는 실무에서 쓰지 않는다. 다대일 - 일대다 - 일대일 순으로 많이 쓴다.

### 단방향, 양방향

테이블은 외래키 하나로 양쪽 조인을 할 수 있어서 사실 방향이라는 개념이 없다. 하지만 객체는 참조용 필드가 있는 쪽으로만 참조할 수 있다. 한 쪽만 참조하면 단방향, 양쪽이 서로
참조하면 양방향이다.

하지만 사실 양방향은 단방향이 2개 존재하는 것일 뿐이다.

### 연관 관계 주인

테이블은 외래키 하나로 두 테이블이 연관 관계를 맺는다. 객체의 양방향 관계는 A->B, B->A처럼 참조가 2곳이 된다.

객체 양방향 관계는 참조가 2개이기 때문에 테이블의 외래키를 관리할 곳을 정해야 한다. 여기서 연관 관계의 주인은 외래키를 관리하는 참조다.

주인의 반대편은 외래 키에 영향을 주지 않는다. 단순 조회만 가능하다.

## 다대일

![](../../.gitbook/assets/kimyounghan-orm-jpa/06/Screen%20Shot%202021-03-20%20at%2012.17.58%20PM.png)

DB 설계 측면에서 보면 당연히 회원이 N, 팀이 1이며 외래키는 N에 가야한다. 안 그러면 설계가 잘못된 것이다. 그래야 멤버를 두 명을 넣어도 `team_id`에 같은 팀을
넣어줄 수 있는 것이다. 반대로 팀에 `member_id`를 넣는다면 팀을 여러 개 insert 해야 하고 그럼 팀이 중복되어 N이 되어버리는 이상한 상황이 된다.

외래키가 있는 곳에 참조를 걸고 연관 관계 매핑을 하면 된다.

### 다대일 단방향 정리

- 가장 많이 사용하는 연관 관계다.
- 다대일의 반대는 일대다이다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/06/Screen%20Shot%202021-03-20%20at%2012.18.06%20PM.png)

양방향으로 설정하는 것이 테이블에 영향을 주는 것은 아니다. 어차피 연관 관계 주인이 외래키를 관리하기 때문에 객체에서 추가만 해주면 된다.

### 다대일 양방향 정리

- 외래키가 있는 쪽이 연관 관계의 주인이다.
- 양쪽을 서로 참조하도록 개발한다.

## 일대다

1:N은 1이 연관 관계의 주인이어서 외래키를 관리하는 것이다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/06/Screen%20Shot%202021-03-20%20at%201.13.02%20PM.png)

우선 이 모델은 권장하지 않는다. 하지만 표준 스펙이기 때문에 설명한다.

팀을 중심으로 뭔가를 해보겠다는 것이다. 팀이 회원을 가지고 있고 회원은 팀을 알고 싶지 않은 것이다. DB 입장에서는 무조건 N 쪽에 외래키가 들어가야 한다. 하지만 이
상황에서는 팀의 회원 값을 바꾸면 `Member`라는 다른 테이블에 있는 외래키 `team_id`를 업데이트 해야 하는 것이다.

{% endtab %} {% tab title="Team.java" %}

```java

@Entity
public class Team {

  @Id
  @GeneratedValue(strategy = GenerationType.AUTO)
  @Column(name = "TEAM_ID")
  private Long id;

  @OneToMany
  @JoinColumn(name = "TEAM_ID")
  private List<Member> members = new ArrayList<>();
}

```

{% endtab %} {% tab title="Member.java" %}

```java

@Entity
public class Member {

  // id랑 username만 있고 team은 없다.
  @Id
  @GeneratedValue(strategy = GenerationType.AUTO)
  private Long id;
  private String name;
}

```

{% endtab %} {% tab title="JpaMain.java" %}

```java
public class JpaMain {

  public static void main(String[] args) {
    Member member = new Member();
    member.setUsername("member1");

    em.persist(member);

    // 이 내용은 팀 테이블에 그냥 넣으면 된다.
    Team team = new Team();
    team.setName("teamA");
    // 그런데 이 데이터는 팀 테이블이 아니라 
    // 외래 키가 있는 멤버 테이블로 가서 업데이트 해야 한다.
    team.getMembers().add(member);

    em.persist(team);

    tx.commit();
  }
}

```

{% endtab %}{% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/06/Screen%20Shot%202021-03-20%20at%201.32.42%20PM.png)

회원과 팀에 대해 정상적으로 insert 쿼리가 나갔다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/06/Screen%20Shot%202021-03-20%20at%201.33.01%20PM.png)

그 뒤에 회원에 대해 update 쿼리가 추가로 나간 것을 볼 수 있다.

JPA를 잘 모르면 비즈니스 로직만 봤을 땐 팀만 손을 댄 것 같은데 쿼리를 보니 엉뚱한 회원 update 쿼리가 나가서 혼란이 생긴다. 또한, 실무에서는 몇 십개의 테이블이
돌아가기 때문에 더 다루기 힘들어진다.

사실 객체적으로만 보면 회원이 팀을 가지고 있을 이유가 없을 수 있다. 하지만 다대일로 하면 객체 지향으로는 손해를 보더라도(멤버에서 팀으로 갈 일이 없더라도) 잘 사용할 수 있다. DB의 방향에 조금 맞춤으로써 유지 보수를 호율적으로 할 수 있다.

### 일대다 단방향 정리

- 1:N에서 1이 연관 관계의 주인이다.
- 테이블 일대다 관계는 항상 N에 외래키가 있다.
- 객체와 테이블의 차이 때문에 반대편 테이블의 외래키를 관리하는 특이한 구조가 된다.
- @JoinColumn을 꼭 사용해야 한다.
    - 대신에 조인 테이블 방식으로 중간에 테이블을 하나 추가하는 방법도 있다.
- 엔티티가 고나리하는 외래키가 다른 테이블에 있어 update SQL을 추가로 실행해야 하는 단점이 있다.
- 일대다 단방향 매핑보다는 다대일 양방향 매핑을 사용하자.

![](../../.gitbook/assets/kimyounghan-orm-jpa/06/Screen%20Shot%202021-03-20%20at%201.13.08%20PM.png)

{% endtab %} {% tab title="Team.java" %}

```java

@Entity
public class Team {

  @Id
  @GeneratedValue(strategy = GenerationType.AUTO)
  @Column(name = "TEAM_ID")
  private Long id;

  @OneToMany
  @JoinColumn(name = "TEAM_ID")
  private List<Member> members = new ArrayList<>();
}

```

{% endtab %} {% tab title="Member.java" %}

```java

@Entity
public class Member {

  @Id
  @GeneratedValue(strategy = GenerationType.AUTO)
  private Long id;
  private String name;
  
  @ManyToOne
  @JoinColumn(name = "TEAM_ID", insertable = false, updateble = false)
  private Team team;
}

```

{% endtab %} {% endtabs %}

이렇게 되면 `Team`의 `members`도, `Member`의 `team`도 연관 관계의 주인이 되어버린다. 어디를 기준으로 할지 모르게 되는 것이다. 그래서 `insertable`과 `updatable`를 넣어서 읽기 전용으로 강제한다.

### 일대다 양방향 정리

- 이런 매핑은 공식적으로 존재하지 않는다.
- 읽기 전용 필드를 사용해 양방향처럼 사용한다.
- 그냥 다대일 양방향을 사용하자.

