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

{% endtab %} {% tab title="JpaMain.java" %}

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

## 고아 객체

{% tabs %} {% tab title="Parent.java" %}

```java

@Entity
public class Parent {

  @Id
  @GeneratedValue
  private Long id;

  // 고아 객체 옵션을 준다.
  @OneToMany(mappedBy = "parent", orphanRemoval = true)
  private List<Child> childList = new ArrayList<>();

  public void addChild(Child child) {
    childList.add(child);
    child.setParent(this);
  }
}
```

{% endtab %} {% tab title="JpaMain.java" %}

```java
public class JpaMain {

  public static void main(String[] args) {
    Parent parent = em.find(Parent.class, id);
    // 자식 엔티티를 컬렉션에서 제거한다. 즉, 연관 관계를 끊는다.
    parent.getChildren().remove(0);

    tx.commit();
  }
}
```

{% endtab %} {% endtabs %}

`orphanRemoval = true` 옵션을 주면, 부모 엔티티와 연관 관계가 끊어진 자식 엔티티를 자동으로 삭제한다. 연관 관계를 끊었을 때 `delete from child where id = ?` 쿼리가 자동으로 나간다.

### 주의 사항

- 특정 엔티티가 개인 소유할 때 사용한다.
- `@OneToOne`, `@OneToMany`만 가능하다.

`orphanRemoval = true`는 엔티티의 참조가 제거되면 그 객체를 다른 곳에서 참조하지 않는 고아 객체로 보고 삭제하는 기능이다. 따라서 참조하는 곳이 하나일 때 사용해야 한다.

{% tabs %} {% tab title="Parent.java" %}

```java

@Entity
public class Parent {

  @Id
  @GeneratedValue
  private Long id;

  @OneToMany(mappedBy = "parent", cascade = CascadeType.REMOVE)
  private List<Child> childList = new ArrayList<>();

  public void addChild(Child child) {
    childList.add(child);
    child.setParent(this);
  }
}
```

{% endtab %} {% tab title="JpaMain.java" %}

```java
public class JpaMain {

  public static void main(String[] args) {
    Parent parent = em.find(Parent.class, id);
    // 자식도 같이 제거된다.
    em.remove(parent);
    
    tx.commit();
  }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/08/screenshot%202021-03-27%20오후%209.45.07.png)

개념적으로 부모를 제거하면 자식은 고아가 된다. 따라서 고아 객체 제거 기능을 활성화 하면, `CascadeType.REMOVE`와 같이 부모를 제거할 때 자식도 함께 제거된다.

## 영속성 전이 + 고아 객체 조합의 생명 주기

`CascadeType.ALL`과 `orphanRemoval = true`를 동시에 사용하면 어떻게 될까?

스스로 생명 주기를 관리하는 엔티티는 em.persist()로 영속화하고 em.remove()로 제거한다. 즉, 라이프 사이클을 JPA 영속성 컨텍스트(엔티티 매니저)를 통해 관리한다.

하지만 두 옵션을 모두 활성화 하면, 부모 엔티티를 통해서 자식의 생명 주기를 관리할 수 있다. 부모만 JPA로 영속화하거나 제거하고 자식은 부모가 관리하는 것이다. 

이 방법은 도메인 주도 설계의 Aggregate Root 개념을 구현할 때 유용하다. 리파지토리는 Aggregate Root만 컨택하고 나머지는 리파지토리를 만들지 않도록 하는 방식이다. 나머지는 Aggregate Root를 통해서 생명주기를 관리한다. 여기서는 Aggregate Root가 parent고 child는 Aggreate Root가 관리한다고 보면 된다.