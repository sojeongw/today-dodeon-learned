# 양방향 연관 관계와 연관 관계의 주인

![](../../.gitbook/assets/kimyounghan-orm-jpa/05/screenshot%202021-03-19%20오후%2010.22.45.png)

양방향으로 한다고 해도 테이블 연관 관계는 변한 것이 없다. 테이블은 member에서 team을, team에서 member를 외래키 하나로 자유롭게 조회할 수 있다.

문제는 객체다. team에 member를 넣어줘야만 접근이 가능하다.

{% endtab %} {% tab title="정의" %}

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

    // Member 엔티티에 있는 team 변수가 연결되어 있음을 설정한다.
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

회원은 팀을 참조하고 팀은 회원을 참조한다. 그럼 둘 중에 뭘로 매핑을 해야할까? 회원의 팀 값을 바꿨을 때 외래키가 업데이트 되어야 할까, 아니면 팀의 회원을 바꿨을 때의 외래키가 업데이트 되어야 할까? 즉, 회원을 새로운 팀에 넣고 싶은데 회원에서 바꿔야할지 팀에서 바꿔야할지 딜레마가 온다.

혹은 회원의 팀에는 값을 넣고 팀에 있는 회원에는 값을 넣지 않았다면 이건 어떻게 처리해야 할까? 테이블에서는 회원에 있는 PK 값만 업데이트 되면 되기 때문에 객체가 문제다.

그래서 둘 중 하나가 외래키를 관리해야 한다. 연관 관계의 주인을 정해야 하는 것이다.

## 양방향 매핑 규칙

- 객체의 두 관계 중 하나를 연관 관계의 주인으로 지정한다.
- 연관 관계의 주인만 외래키 등록, 수정 등을 관리한다.
- 주인이 아닌 쪽은 읽기만 가능하다.
- 주인은 mappedBy 속성을 사용하지 않는다.
- 주인이 아니면 mappedBy 속성으로 주인을 지정한다.

## 주인을 결정하는 기준

![](../../.gitbook/assets/kimyounghan-orm-jpa/05/screenshot%202021-03-19%20오후%2010.54.56.png)

외래키가 있는 곳을 주인으로 정한다. 여기서는 Member.team이 연관 관계의 주인이 된다.
