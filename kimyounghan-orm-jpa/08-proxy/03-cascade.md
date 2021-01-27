# 영속성 전이와 고아 객체

## 영속성 전이

특정 엔티티를 영속 상태로 만들 때 연관된 에티티도 함께 영속 상태로 만든다. 예를 들어, 부모 엔티티를 저장할 떄 자식 엔티티도 함께 저장하는 행위를 말한다.

즉시 로딩, 지연 로딩이나 연관 관계와는 전혀 관계가 없는 개념이다.

{% tabs %} {% tab title="Parent.java" %}

```java

@Entity
public class Parent {

  @Id
  @GeneratedValue
  private Long id;

  @OneToMany(mappedBy = "parent")
  private List<Child> childList = new ArrayList<>();

  public void addChild(Child child) {
    childList.add(child);
    child.setParent(this);
  }
}
```

{% endtab %} {% tab title="Child.java" %}

```java

@Entity
public class Child {

  @Id
  @GeneratedValue
  private Long id;

  @ManyToOne
  @JoinColumn(name = "parent_id")
  private Parent parent;
}
```

{% endtab %} {% tab title="Child.java" %}

```java
public class JpaMain {

  public static void main(String[] args) {
    Child child1 = new Child();
    Child child2 = new Child();

    Parent parent = new Parent();
    parent.addChild(child1);
    parent.addChild(child2);

    // 영속화를 3번 해줘야 한다.
    em.persist(parent);
    em.persist(child1);
    em.persist(child2);

    tx.commit();
  }
}
```

{% endtab %} {% endtabs %}

각각의 엔티티를 영속화 해줘야 insert 쿼리가 날아간다. 이런 반복은 번거로우므로 parent가 저장될 때 child가 같이 저장되도록 해보자.

{% tabs %} {% tab title="Parent.java" %}

```java

@Entity
public class Parent {

  @Id
  @GeneratedValue
  private Long id;

  // cascade 옵션을 준다.
  @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL)
  private List<Child> childList = new ArrayList<>();

  public void addChild(Child child) {
    childList.add(child);
    child.setParent(this);
  }
}
```

{% endtab %} {% tab title="Child.java" %}

```java

@Entity
public class Child {

  @Id
  @GeneratedValue
  private Long id;

  @ManyToOne
  @JoinColumn(name = "parent_id")
  private Parent parent;
}
```

{% endtab %} {% tab title="Child.java" %}

```java
public class JpaMain {

  public static void main(String[] args) {
    Child child1 = new Child();
    Child child2 = new Child();

    Parent parent = new Parent();
    parent.addChild(child1);
    parent.addChild(child2);

    // parent만 영속화하면 자동으로 자식들이 영속화된다.
    em.persist(parent);

    tx.commit();
  }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/08/screenshot%202021-03-26%20오후%203.44.44.png)

parent만 영속화했는데 모두 적용된 것을 볼 수 있다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/08/screenshot%202021-03-26%20오후%203.05.10.png)

부모를 영속화할 때 자식도 적용시키는 것이 `cascade`다.

## 주의 사항

영속성 전이는 연관 관계를 매핑하는 것과 아무 관련이 없다. 엔티티를 영속화할 때 연관된 엔티티도 함께 영속화 하는 편리함을 제공할 뿐이다.

1:N에 다 걸어야 하는 것은 아니다. 자식의 부모가 하나일 때는 써도 된다. 게시판, 첨부파일 경로 같은 경우가 해당된다. 다른 엔티티에서도 관리하는 자식이라면 쓰면 안 된다. 즉, 소유자가 하나일 때는 써도 된다.

## 종류

- ALL
    - 모두 적용
    - 라이프 사이클이 정말 다 동일해야 할 때 사용한다.
- PERSIST
    - 영속
- REMOVE
    - 삭제
- MERGE
    - 병합
- REFRESH
- DETACH

`ALL`, `PERSIST`, `MERGE`를 가장 많이 쓴다.