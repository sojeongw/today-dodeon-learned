# 양방향 연관 관계와 연관 관계의 주인

![](../../.gitbook/assets/kimyounghan-orm-jpa/05/screenshot%202021-03-19%20오후%2010.22.45.png)

- 양방향으로 한다고 해도 테이블 연관 관계는 변한 것이 없다.
    - 테이블은 member에서 team을, team에서 member를 외래키 하나로 join 해서 자유롭게 조회할 수 있다.
- 하지만 객체는 team에 `List<Member> members`를 넣어줘야만 접근이 가능하다.

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
```

{% endtab %} {% tab title="Team.java" %}

```java

@Entity
public class Team {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    // mappedBy는 Member에 있는 'team' 변수가 연결되어 있음을 의미한다.
    // 즉, 반대편에 무엇이 걸려있는지 알려준다.
    @OneToMany(mappedBy = "team")
    List<Member> members = new ArrayList<>(); // 리스트는 초기화 하는 것이 관례다. add 할 때 NPE를 방지한다.

}
```

{% endtab %}{% tab title="조회" %}

```java
public class JpaMain() {

    public static void main(String[] args) {
        ...

        Team findTeam = em.find(Team.class, team.getId());

        // 이제 team에서도 member를 조회할 수 있다.
        int memberSize = findTeam.getMembers().size();
    }
}
```

{% endtab %} {% endtabs %}

## 연관 관계의 주인과 mappedBy

![](../../.gitbook/assets/kimyounghan-orm-jpa/05/screenshot%202021-03-19%20오후%2010.37.12.png)

- 객체는 연관 관계가 2개다.
    - 회원에서 팀으로
    - 팀에서 회원으로
    - 단방향이 2개 있는 것이다.
- 테이블은 연관 관계가 1개다.
    - 회원과 팀 사이의 양방향
    - member 테이블의 FK와 team의 PK를 조인하면 된다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/05/screenshot%202021-03-19%20오후%2010.37.17.png)

- 테이블 연관 관계는 FK 하나로 끝이 나지만, 객체는 참조가 두 곳에 다 있어야 한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/05/screenshot%202021-03-19%20오후%2010.37.26.png)

- 객체의 양방향 관계는 사실 양방향이 아니라, 서로 다른 단방향 관계가 2개인 상태다.
- 객체를 양방향으로 참조하려면 단방향 연관 관계를 2개 만들어야 한다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/05/screenshot%202021-03-19%20오후%2010.37.31.png)

- 하지만 테이블은 외래키 하나로 두 테이블의 연관 관계를 관리한다.
- 외래키 하나로 양방향 연관 관계를 갖고 있으므로 양쪽으로 조인할 수 있다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/05/screenshot%202021-03-19%20오후%2010.37.37.png)

- member를 새로운 team에 넣고 싶은데 member에서 team 값을 바꿔야할지 team에서 members 값을 바꿔야할지 딜레마가 온다.
    - member의 team 값을 수정했을 때 외래키 값이 업데이트 되어야 할까,
    - 아니면 team의 members 값을 수정했을 때의 외래키 값이 업데이트 되어야 할까?
- DB 입장에서는 어찌됐든 member에 있는 FK인 team_id만 업데이트 되면 된다.
- 결국 둘 중 하나만 외래키를 관리해야 한다. 연관 관계의 주인을 정해야 하는 것이다.
    - 연관 관계의 주인은 양방향 매핑에서만 나온다.

## 연관 관계의 주인

- 객체의 두 관계 중 하나를 연관 관계의 주인으로 지정한다.
- 연관 관계의 주인
    - 외래키 등록, 수정 등을 관리한다.
    - mappedBy 속성을 사용하지 않는다.
- 주인이 아닌 쪽
    - 읽기만 가능하다.
    - mappedBy 속성을 사용해 주인이 누군지 명시한다.

### 주인을 결정하는 기준

```java

@Entity
public class Member {
    ...

    // 외래키가 있는 곳을 연관 관계의 주인으로 정한다.
    // 연관 관계 주인인 team만 insert, update를 할 수 있다.
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;
}

@Entity
public class Team {
    ...

    // team에 의해 관리가 된다는 뜻이다. 즉, team이 연관 관계의 주인이다.
    // 따라서 읽기만 가능하다. 값을 넣어도 아무 일이 일어나지 않는다.
    @OneToMany(mappedBy = "team")
    List<Member> members = new ArrayList<>();
}
```

![](../../.gitbook/assets/kimyounghan-orm-jpa/05/screenshot%202021-03-19%20오후%2010.54.56.png)

- 주인
    - 외래키가 있는 곳인 Many를 주인으로 정한다.
    - Member.team
- 가짜 매핑
    - 주인의 반대편인 One 쪽을 말한다.
    - Team.members

Member.team이 주인이 되면 외래키가 Member에 있기 때문에 쿼리를 한 방에 보낼 수 있다. Member 객체를 바꿨으니까 member 테이블에 업데이트 쿼리가 나가는구나 하고 직관적으로 이해가 된다.

반대로 Team.members가 주인이 되면, members를 바꿨을 때 내가 수정한 테이블인 team이 아니라 member 테이블에 쿼리가 나가야 한다. 나는 team 객체를 수정했는데 member 테이블이라는
엉뚱한 곳에 쿼리가 나가기 때문에 혼란스럽다.

### Tip

- 헷갈리면 무조건 외래키가 있는 곳을 주인으로 정하면 된다.
- DB 입장에서 보면 외래키가 있는 곳이 무조건 N이고 외래키가 없는 곳은 무조건 1이다.
- 즉, N쪽이 무조건 연관 관계의 주인이 되는 것이다.
    - N쪽이 무조건 `ManyToOne`이 된다.

연관 관계의 주인이라고 하면 뭔가 비즈니스적으로 중요하게 느껴지지만 그것과는 전혀 상관이 없다. 그냥 N쪽인 곳이 주인이 되면 된다.

자동차와 자동차 바퀴로 비유를 하자면, 비즈니스적으로는 자동차가 중요하지만 연관 관계의 주인은 바퀴가 되어야 한다. N쪽이 연관 관계의 주인이 되면 된다.

## 양방향 매핑 시 가장 많이 하는 실수

### 연관 관계 주인에 값을 입력하지 않음

```java
public class JpaMain {

    public static void main(String[] args) {
        ...

        // member를 하나 만든다.
        Member member = new Member();
        member.setName("member1");

        em.persist(member);

        // team에 만든 member를 추가한다.
        Team team = new Team();
        team.setName("TeamA");
        team.getMembers().add(member);

        em.persist(team);
    }
}
```

```text
insert into Member(team_id, username, member_id) values (?, ?, ?)
insert into Team(name, team_id) values (?, ?)
```

![](../../.gitbook/assets/kimyounghan-orm-jpa/05/screenshot%202021-03-19%20오후%2011.20.39.png)

- insert 쿼리는 분명 2번이 나갔지만 회원에 팀 아이디가 저장되지 않았다.
- JPA는 insert나 update할 때 읽기 전용 필드를 아예 보지 않기 때문이다.
    - 연관 관계의 주인은 Member.team이다.
    - Team.members는 mappedBy 된 읽기 전용이다.
    - Team.members에서 수정을 했으니 반영되지 않은 것이다.

```java
public class JpaMain {

    public static void main(String[] args) {
        Team team = new Team();
        team.setName("TeamA");
        // team에 member를 추가하는 대신
        // team.getMembers().add(member);

        em.persist(team);

        Member member = new Member();
        member.setName("member1");
        // 연관 관계의 주인인 Member.team에 추가한다.
        member.setTeam(team);

        em.persist(member);
    }
}
```

- 연관 관계의 주인인 곳에만 값을 넣도록 수정하면 정상 반영된다.

### 양쪽에 모두에 값을 세팅하지 않음

```java
public class JpaMain {

    public static void main(String[] args) {
        Team team = new Team();
        team.setName("TeamA");

        em.persist(team);

        Member member = new Member();
        member.setName("member1");
        // 연관 관계 주인에 세팅
        member.setTeam(team);

        em.persist(member);

        em.flush();
        em.clear();

        // Team.members에 세팅해준 게 없어도 select 쿼리를 치면 데이터가 나온다.
        Team findTeam = em.find(Team.class, team.getId());
        List<Member> members = findTeam.getMembers();

        for (Member m : members) {
            System.out.println("m = " + m.getUsername());
        }
    }
}
```

- 주인 쪽에만 세팅해도 반대편에 같이 적용되기 때문에 findTeam.getMembers()로 조회하면 select 쿼리가 나간다.

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

        // 하지만 양쪽 다 연관 관계 설정를 추가하는 게 좋다.
        team.getMembers().add(member);

        // 만약 flush와 clear를 해주지 않으면
//        em.flush();
//        em.clear();

        // team은 em.persist(team) 해줬을 때의 상태 그대로 1차 캐시에 들어가있다.
        // 연관 관계가 적용된 데이터는 메모리에만 올라가 있다.
        // 따라서 여기서 Team.members를 조회하면 값이 나오지 않는다.
        // 영속성 컨텍스트에는 team.setName()까지만 했던 그대로 있기 때문이다.
        Team findTeam = em.find(Team.class, team.getId());
        // 따라서 getMembers()를 하면 select 쿼리가 나가지 않는다.
        List<Member> members = findTeam.getMembers();

        for (Member m : members) {
            System.out.println("m = " + m.getUsername());
        }
    }
}
```

- 하지만 양쪽 다 연관 관계를 설정해주는 게 좋다.
    - flush(), clear()가 없으면 연관 관계가 반영되지 않은 데이터를 1차 캐시에서 가져올 수 있기 때문이다.
    - 객체 지향적으로도 순수 객체 상태를 고려해 항상 양쪽에 값을 설정 해주는 게 맞다.
    - JPA 없이 순수하게 동작하도록 테스트 케이스를 짰을 경우도 team.getMembers()가 빈 값으로 나오는 문제가 발생할 수 있다.

## 연관 관계 편의 메서드

{% tabs %} {% tab title="Member.java" %}

```java
public class Member {
    ...

    // 연관 관계 메서드
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
        // 연관 관계 편의 메서드
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

- 연관 관계 편의 메서드를 사용하면 양쪽에 설정하는 까먹지 않을 수 있다.
- setTeam()에서 Member.team을 세팅할 때 member 자신도 Team.members()에 세팅한다.
- 메서드를 원자적으로 즉, 하나만 써도 양쪽으로 적용할 수 있다.

{% tabs %} {% tab title="Team.java" %}

```java
public class Team {
    ...

    // 가짜 매핑쪽에 만든 연관 관계 편의 메서드
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

        // 연관 관계 편의 메서드
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

- 연관 관계 편의 메서드는 반대로도 할 수 있다.
- 양쪽으로 설정하면 문제가 되므로 무엇이 기준이 될지 정하자.
- 연관 관계 주인은 계속 회원에 있는 팀이지만 값을 세팅하는 것은 자유다.
    - 중요한 건 둘 중 하나에만 해야한다는 것이다.

## 무한 루프

- toString(), lombok, JSON 생성 라이브러리 등에서 문제가 된다.

```java
public class Member {

    @Override
    public String toString() {
        return "Member{" +
                "id=" + id +
                ", name='" + name + '\'' +
                // 무한 루프
                ", team='" + team + '\'' +
                '}';
    }
}
```

- toString()
    - team을 출력한다.
    - team.toString()도 호출한다.
    - 그런데 team에서도 toString()에서 member를 호출한다.
    - 무한 루프에 빠진다.
- JSON 라이브러리
    - 엔티티의 연관 관계가 양방향일 때
    - 컨트롤러에서 response로 엔티티를 직접 보내는 경우
    - JSON으로 변환할 때 무한 루프에 빠진다.

`toString()`은 웬만하면 쓰지 말자.

JSON 라이브러리의 경우, 컨트롤러에서 엔티티 대신 DTO로 변환해서 보내면 된다. 엔티티를 바로 반환하면 무한 루프 이슈뿐 아니라 엔티티를 변경하는 순간 API 스펙이 바뀌어 버리는 문제가 있다.

## 정리

- 웬만하면 단방향 매핑으로 끝내야 한다.
    - 단방향 매핑만으로도 이미 연관 관계 매핑은 완료된다.
    - 테이블 설계를 어느 정도 하면서 객체 설계를 하므로 테이블에서 파악된 FK로 단방향 매핑을 설계해야 한다.
- 양방향 매핑은 반대 방향으로 조회(객체 그래프 탐색)하는 기능을 추가한 것 뿐이다.
    - 오히려 양방향을 하면 고려할 것들만 많아진다.
        - 연관 관계 편의 메서드도 만들어야 한다.
- 실무에서 JPQL 등으로 역방향 참조가 필요한 경우 그때 추가하면 된다.
    - 단방향 매핑을 잘 해놓은 다음, 양방향은 필요할 때 사용한다.
    - 테이블에 영향가는 것 없이 연관 관계만 설정해주면 되기 때문이다.
- 연관 관계의 주인
    - 비즈니스 로직 기준 X
    - 외래 키의 위치 기준 O