# 양방향 연관 관계와 연관 관계의 주인

![](../../.gitbook/assets/kimyounghan-orm-jpa/05/screenshot%202021-03-19%20오후%2010.22.45.png)

양방향으로 한다고 해도 테이블 연관 관계는 변한 것이 없다. 테이블은 member에서 team을, team에서 member를 외래키 하나로 자유롭게 조회할 수 있다.

문제는 객체다. team에 member를 넣어줘야만 접근이 가능하다.

{% tabs %} {% tab title="정의" %}

```java

@Entity
public class Member {

  @Id
  @GeneratedValue
  private Long id;

  @Column(name = "USERNAME")
  private String name;
  private int age;

  @ManyToOne
  @Column(name = "TEAM_ID")
  private Team team;
}

@Entity
public class Team {

  @Id
  @GeneratedValue
  private Long id;

  private String name;

  // Member Entity에 있는 team 변수가 연결되어 있음을 설정한다.
  // 즉, 반대편에 무엇이 걸려있는지 알려준다.
  @OneToMany(mappedBy = "team")
  // 초기화 하는 것이 관례다. add할 때 npe이 뜨지 않게 하기 위함이다.
      List<Member> members = new ArrayList<Member>();
}
```

{% endtab %} {% tab title="조회" %}

```java
public class JpaMain() {

  public static void main(String[] args) {
    Team findTeam = em.find(Team.class, team.getId());
    // 이제 반대로도 조회할 수 있다.
    int memberSize = findTeam.getMembers().size();
  }
}
```

{% endtab %} {% endtabs %}

## 연관 관계의 주인과 mappedBy

![](../../.gitbook/assets/kimyounghan-orm-jpa/05/screenshot%202021-03-19%20오후%2010.37.12.png)

![](../../.gitbook/assets/kimyounghan-orm-jpa/05/screenshot%202021-03-19%20오후%2010.37.17.png)

객체는 회원에서 팀으로, 팀에서 회원으로의 두 가지 연관 관계가 있다. 단방향 연관 관계가 2개 있는 것이다.

테이블은 연관 관계가 1개다. PK로 조인하면 양방향으로 조회할 수 있다.

테이블 연관 관계는 FK 하나로 끝이 나지만, 객체는 참조가 두 곳에 다 있어야 한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/05/screenshot%202021-03-19%20오후%2010.37.26.png)

앞서 말했듯, 객체의 양방향 관계는 사실 양방향이 아니라 서로 다른 단방향 관계가 2개인 상태다. 객체를 양방향으로 참조하려면 단방향 연관 관계를 2개 만들어야 한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/05/screenshot%202021-03-19%20오후%2010.37.31.png)

하지만 테이블은 외래키 하나로 두 테이블의 연관 관계를 관리한다. 외래키 하나로 양방향 연관 관계를 가진다. 즉, 양쪽으로 조인할 수 있다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/05/screenshot%202021-03-19%20오후%2010.37.37.png)

회원은 팀을 참조하고 팀은 회원을 참조한다. 그럼 둘 중에 뭘로 매핑을 해야할까? 회원의 팀 값을 바꿨을 때 외래키가 업데이트 되어야 할까, 아니면 팀의 회원을 바꿨을 때의
외래키가 업데이트 되어야 할까? 즉, 회원을 새로운 팀에 넣고 싶은데 회원에서 바꿔야할지 팀에서 바꿔야할지 딜레마가 온다.

혹은 회원의 팀에는 값을 넣고 팀에 있는 회원에는 값을 넣지 않았다면 이건 어떻게 처리해야 할까? 테이블에서는 회원에 있는 PK 값만 업데이트 되면 되기 때문에 객체가 문제다.

그래서 둘 중 하나가 외래키를 관리해야 한다. 연관 관계의 주인을 정해야 하는 것이다.

## 양방향 매핑 규칙

- 객체의 두 관계 중 하나를 연관 관계의 주인으로 지정한다.
- 연관 관계의 주인만 외래키 등록, 수정 등을 관리한다.
- 주인이 아닌 쪽은 읽기만 가능하다.
- 주인은 mappedBy 속성을 사용하지 않는다.
- 주인이 아니면 mappedBy 속성으로 주인을 지정한다.

## 주인을 결정하는 기준

```java

@Entity
public class Member {
    ...

  // 연관 관계의 주인
  // insert, update를 할 때는 team만 참조할 수 있다.
  @ManyToOne
  @Column(name = "TEAM_ID")
  private Team team;
}

@Entity
public class Team {
    ...

  // team에 의해 관리가 된다는 뜻이다.
  // 즉, team이 연관 관계의 주인이다.
  // 따라서 읽기만 가능하다.
  // 값을 넣어도 아무 일이 일어나지 않는다.
  @OneToMany(mappedBy = "team")
  List<Member> members = new ArrayList<Member>();
}
```

![](../../.gitbook/assets/kimyounghan-orm-jpa/05/screenshot%202021-03-19%20오후%2010.54.56.png)

외래키가 있는 곳을 주인으로 정한다. 여기서는 `Member.team`이 연관 관계의 주인이 된다.

Many 쪽이 연관 관계의 주인이 되는 것이다. 이때 주인의 반대편인 One 쪽 즉, `Team.members`를 가짜 매핑이라고 한다.

회원 객체랑 테이블을 매핑했을 때 외래키도 결국 회원에 있다. 따라서 회원을 연관 관계의 주인이 되어야 한다.

만약 반대로 한다면, 팀의 회원을 바꿨을 때 팀이 아니라 회원 테이블에 쿼리가 나가야 한다. 이 자체가 일단 혼란을 초래한다. 성능 이슈도 있다.

회원이 주인이 되면 회원은 외래키를 한 번에 챙길 수 있기 때문에 회원 테이블에 한 방에 쿼리를 때린다. 팀이 주인이 되면 팀에는 insert 쿼리가 나가는데 회원에는 update
쿼리가 나간다.

DB 입장에서 보면 외래키가 있는 곳이 무조건 N이다. 외래키가 없는 곳은 무조건 1이다. 즉 N쪽이 무조건 연관 관계의 주인이 되는 것이다. 따라서 N쪽이
무조건 `ManyToOne`이 된다.

연관 관계의 주인이라고 하면 뭔가 비즈니스적으로 중요하게 느껴지지만 그것과는 전혀 상관이 없다. 그냥 N쪽인 곳이 주인이 되면 된다. 자동차와 자동차 바퀴로 비유를 하자면,
비즈니스적으로는 자동차가 중요하지만 연관 관계의 주인은 바퀴가 되어야 한다. N쪽이 연관 관계의 주인이 되면 된다.

## 양방향 매핑 시 가장 많이 하는 실수

### 연관 관계 설정

```java
public class JpaMain {

  public static void main(String[] args) {
    Member member = new Member();
    member.setName("member1");
    em.persist(member);
      
    Team team = new Team();
    team.setName("TeamA");
    team.getMembers().add(member);
    em.persist(team);
  }
}
```

![](../../.gitbook/assets/kimyounghan-orm-jpa/05/screenshot%202021-03-19%20오후%2011.20.39.png)

insert 쿼리는 분명 2번이 나갔지만 회원에 팀 아이디가 저장되지 않았다. 연관 관계의 주인은 회원 클래스에 있는 팀이다. `team.getMembers()`
는 `mappedBy`된 필드이기 때문에, JPA에서 insert나 update할 때는 이 필드를 아예 보지 않는다. 그냥 읽기 전용인 가짜 매핑이다.

```java
public class JpaMain {

  public static void main(String[] args) {
    Team team = new Team();
    team.setName("TeamA");
    // team에 member를 추가하는 게 아니라
//        team.getMembers().add(member);
    em.persist(team);

    Member member = new Member();
    member.setName("member1");
    // member에 team을 추가한다.
    member.setTeam(team);

    em.persist(member);

  }
}
```

그래서 위와 같이 연관 관계의 주인인 곳에만 값을 넣도록 수정해야 한다.

```java
public class JpaMain {

  public static void main(String[] args) {
    Team team = new Team();
    team.setName("TeamA");
    em.persist(team);

    Member member = new Member();
    member.setName("member1");
    member.setTeam(team);
    em.persist(member);

    em.flush();
    em.clear();

    Team findTeam = em.find(Team.class, team.getId());
    List<Member> members = findTeam.getMembers();

    for (Member m : members) {
      System.out.println("m = " + m.getUsername());
    }
  }
}
```

주인 쪽에만 값을 세팅해줘도 `team.getMembers()`로 조회하면 select 쿼리가 나간다. 하지만 사실 순수한 객체 관계를 고려하면, 항상 양쪽 다 값을 입력해야
한다.

```java
public class JpaMain {

  public static void main(String[] args) {
    Team team = new Team();
    team.setName("TeamA");
    em.persist(team);

    Member member = new Member();
    member.setName("member1");
    member.setTeam(team);
    em.persist(member);

//        team.getMembers().add(member);

//        em.flush();
//        em.clear();

    // 1차 캐시에 있는 상태
    // 위에서 team.setName() 해줬을 때의 상태 그대로
    // 영속성 컨텍스트에 들어가있다.
    // 그냥 메모리에 올라가있는 것이다.
    Team findTeam = em.find(Team.class, team.getId());
    // 따라서 여기서 조회를 하면 member 값이 나오지 않는다.
    // 영속성 컨텍스트에는 team.setName()까지만 했던 그대로 있기 때문이다.
    // 따라서 select 쿼리가 나가지 않는다.
    List<Member> members = findTeam.getMembers();

    for (Member m : members) {
      System.out.println("m = " + m.getUsername());
    }
  }
}
```

캐시 문제 때문에 양쪽에 설정해주는 것이 맞다. 특히 테스트 케이스를 작성할 때는 JPA 없이도 동작해야 하기 때문에 양쪽으로 설정하는 것이 필요하다.

`flush()`와 `clear()`를 하면 캐시를 날려서 DB를 다시 조회했지만 순수한 1차 캐시에 들어간 상태에서는 문제가 발생한다. 따라서 순수 객체 상태를 고려해 항상
양쪽에 값을 설정하자.

{% tabs %} {% tab title="Member.java" %}

```java

@Entity
public class Member {
    ...

  @ManyToOne
  @Column(name = "TEAM_ID")
  private Team team;

  // 연관 관계 메서드
  public void setTeam(Team team) {
    this.team = team;
    team.getMembers().add(this);
  }
}
```

{% endtab %} {% tab title="JpaMain.java" %}

```java
public class JpaMain {

  public static void main(String[] args) {
    Team team = new Team();
    team.setName("TeamA");
    em.persist(team);

    Member member = new Member();
    member.setName("member1");
    member.setTeam(team);
    em.persist(member);

    em.flush();
    em.clear();

    Team findTeam = em.find(Team.class, team.getId());
    List<Member> members = findTeam.getMembers();

    for (Member m : members) {
      System.out.println("m = " + m.getUsername());
    }
  }
}
```

{% endtab %}{% endtabs %}

그런데 사람인지라 양쪽으로 세팅하는 것을 까먹을 때가 있다. 그래서 연관 관계 메서드를 생성하는 것이 좋다.

위와 같이 `setTeam()`을 할 때 자신도 팀의 회원에 넣어주도록 하면 놓치지 않을 수 있다. 메서드를 원자적으로 즉, 하나만 써도 양쪽으로 적용되도록 사용할 수 있는
것이다.

{% tabs %} {% tab title="Member.java" %}

```java
public class Member {
    ...

  public void changeTeam(Team team) {
    this.team = team;
    team.getMembers().add(this);
  }
}
```

{% endtab %} {% tab title="JpaMain.java" %}

```java
public class JpaMain {

  public static void main(String[] args) {
    Team team = new Team();
    team.setName("TeamA");
    em.persist(team);

    Member member = new Member();
    member.setName("member1");
    // 연관 관계 메서드
    member.changeTeam(team);
    em.persist(member);

    em.flush();
    em.clear();

    Team findTeam = em.find(Team.class, team.getId());
    List<Member> members = findTeam.getMembers();

    for (Member m : members) {
      System.out.println("m = " + m.getUsername());
    }
  }
}
```

{% endtab %}{% endtabs %}

다만, `set()`은 로직이 없는 부분에만 관례상 사용하므로 메서드 이름을 `changeTeam()`으로 바꿔준다. 그러면 이 메서드가 단순히 setter가 아니라 중요한 역할을
하고 있음을 알게 된다.

{% tabs %} {% tab title="Team.java" %}

```java
public class Team {
    ...

  public void addMember(Member member) {
    member.setTeam(this);
    members.add(member);
  }
}
```

{% endtab %} {% tab title="JpaMain.java" %}

```java
public class JpaMain {

  public static void main(String[] args) {
    Team team = new Team();
    team.setName("TeamA");
    em.persist(team);

    Member member = new Member();
    member.setName("member1");
    em.persist(member);

    // 연관 관계 메서드
    team.addMember(member);

    em.flush();
    em.clear();

    Team findTeam = em.find(Team.class, team.getId());
    List<Member> members = findTeam.getMembers();

    for (Member m : members) {
      System.out.println("m = " + m.getUsername());
    }
  }
}
```

{% endtab %}{% endtabs %}

연관 관계 메서드는 이렇게 반대로도 할 수 있다. 양쪽으로 설정하면 문제가 되므로 무엇이 기준이 될지 정하자. 연관 관계 주인은 계속 회원에 있는 팀이지만 값을 세팅하는 것은
자유이기 때문에 자유롭게 쓰면 된다. 중요한 건 둘 중 하나에만 해야한다는 것이다.

### 무한 루프

`toString()`, `lombok`, JSON 생성 라이브러리에서 문제가 된다.

```java
public class Member {

  @Override
  public String toString() {
    return "Member{" +
        "id=" + id +
        ", name='" + name + '\'' +
        ", team='" + team + '\'' +
        '}';
  }
}
```

`toString()`을 보면 `team`을 출력하고 있다. 이 말은 `team.toString()`도 호출한다는 뜻이다. 하지만 팀에 가면 팀에서도 `toString()`
에서 `member`를 호출해서 무한 루프가 돌게 된다.

JSON 라이브러리도 쭉 뽑아버리기 때문에 무한 루프가 돌면서 장애가 난다. 예를 들어 Entity를 직접 컨트롤러에서 response로 보내버리면 Entity가 가진 연관 관계가 양방향일
때 JSON으로 바뀌면서 무한 루프를 도는 것이다.

따라서 `toString()`은 웬만하면 쓰지 말자.

JSON 라이브러리는 컨트롤러에서 Entity를 반환하지 않도록 하면 된다. Entity를 바로 반환하면 무한 루프 이슈뿐 아니라 Entity를 변경하는 순간 API 스펙이 바뀌어 버리는 문제가
있다. 따라서 Entity 대신 DTO로 변환해서 반환하는 것을 추천한다.

## 정리

단방향 매핑만으로도 이미 연관 관계 매핑은 완료된다. 처음에 JPA를 설계할 때 웬만하면 단방향 매핑으로 끝내야 한다. 테이블 설계를 어느정도 하면서 객체 설계를 하므로 테이블에서
파악된 FK로 단방향 매핑을 설계해야 한다.

양방향 배핑은 반대 방향으로 조회(객체 그래프 탐색) 기능이 추가된 것 뿐이다. 따라서 DB와 JPA의 설계는 단방향 매핑만으로 이미 완료가 되는 것이다.

오히려 양방향을 하면 고려할 것들만 많아진다. 연관 관계 편의 메서드도 만들어야 하고. 테이블과 객체의 매핑이라는 설계의 관점에서만 보면 단방향으로만 끝나야 한다.

나중에 실무에서 개발하다 보면 JPQL 등을 사용해 역방향으로 참조해야 할 경우가 생긴다. 단방향 매핑을 잘 해놓은 다음 양방향은 필요할 때 추가하면 된다. 테이블에 손 댈 필요
없이 연관 관계만 설정해주면 되기 때문이다.

연관 관계의 주인은 비즈니스 로직이 아니라 외래키의 위치를 기준으로 정해야 한다. DB 테이블을 설계하고 나니 여기에 PK가 들어간다? 그러면 그걸 연관 관계로 하면 되는 것이다.
굳이 `@OneToMany`가 선언된 부분에 뭔갈 하고 싶다면 연관 관계 편의 메서드를 활용한다.