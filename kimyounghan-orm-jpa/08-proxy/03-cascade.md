# 영속성 전이와 고아 객체

## 영속성 전이

- 특정 Entity를 영속 상태로 만들 때 연관된 Entity도 함께 영속 상태로 만든다.
    - ex. 부모 Entity를 저장할 때 자식 Entity도 함께 저장한다.
- 즉시 로딩, 지연 로딩, 연관관계 세팅과 전혀 관계 없는 개념

{% tabs %} {% tab title="Parent.java" %}

```java

@Entity
public class Parent {

    @Id
    @GeneratedValue
    private Long id;

    @OneToMany(mappedBy = "parent")
    private List<Child> childList = new ArrayList<>();

    // 양방향 연관 관계를 만들어 주기 위한 편의 메서드
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

        // 부모, 자식 각각 영속화 하면 총 3번 해줘야 해서 번거롭다.
        em.persist(parent);
        em.persist(child1);
        em.persist(child2);

        tx.commit();
    }
}
```

{% endtab %} {% endtabs %}

- 각각의 Entity를 영속화 해줘야 insert 쿼리가 날아간다.
- 이런 반복은 번거로우므로 parent가 저장될 때 child가 같이 저장되도록 해보자.

{% tabs %} {% tab title="Parent.java" %}

```java

@Entity
public class Parent {

    @Id
    @GeneratedValue
    private Long id;

    // cascade 옵션을 주면 영속화가 자식에게도 적용된다.
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

- parent만 영속화했는데 모두 적용되었다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/08/screenshot%202021-03-26%20오후%203.05.10.png)

- 부모를 영속화할 때 자식도 적용시키는 것이 `cascade`다.

## 주의 사항

- 영속성 전이는 연관 관계를 매핑하는 것과 아무 관련이 없다.
- Entity를 영속화할 때 연관된 Entity도 함께 영속화 하는 편리함을 제공할 뿐이다.
- 자식의 부모가 하나일 때, 단일 엔티티에 완전히 종속적이고 라이프 사이클이 같을 때 사용한다.
    - ex. 게시판, 첨부파일 경로
- 다른 Entity에서도 관리하는 자식이라면 쓰면 안 된다.

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

## 고아 객체 제거

- 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제한다.

### orphanRemoval = true

{% tabs %} {% tab title="Parent.java" %}

```java

@Entity
public class Parent {

    @Id
    @GeneratedValue
    private Long id;

    // 고아 객체 옵션을 주면 부모가 삭제됐을 때 자식 엔티티를 컬렉션에서 삭제한다.
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
        // 자식 Entity를 컬렉션에서 제거한다. 즉, 연관 관계를 끊는다.
        // 그럼 그 Entity를 삭제한다.
        parent.getChildren().remove(0);

        tx.commit();
    }
}
```

{% endtab %} {% endtabs %}

- 부모 Entity와 연관 관계가 끊어진 자식 Entity를 자동으로 삭제한다.
    - Entity의 참조가 제거되면 그 객체를 다른 곳에서 참조하지 않는 고아 객체로 보고 삭제한다.
    - 따라서 **참조하는 곳이 하나일 때** 사용해야 한다.
- 연관 관계를 끊었을 때 `delete from child where id = ?` 쿼리가 자동으로 나간다.
- `@OneToOne`, `@OneToMany`만 가능하다.

### CascadeType.REMOVE

{% tabs %} {% tab title="Parent.java" %}

```java

@Entity
public class Parent {

    @Id
    @GeneratedValue
    private Long id;

    // cascade에 remove 옵션을 준다.
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
        // 부모를 삭제하면 자식도 같이 삭제한다.
        em.remove(parent);

        tx.commit();
    }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-orm-jpa/08/screenshot%202021-03-27%20오후%209.45.07.png)

- 부모를 제거할 때도 자식은 고아가 되므로 orphanRemoval = true라면 자식도 제거된다.
    - CascadeType.REMOVE처럼 동작한다.

## `CascadeType.ALL`과 `orphanRemoval = true` 동시 사용

- Entity는 스스로 생명 주기를 관리한다.
    - JPA 영속성 컨텍스트(EntityManager)를 통해 라이프 사이클을 관리한다.
        - em.persist()로 영속화하고 em.remove()로 제거한다.
- 두 옵션을 모두 활성화 하면, 부모 Entity를 통해서 자식의 생명 주기를 관리할 수 있다.
    - 부모만 JPA로 영속화하거나 제거하고 자식은 부모가 관리하게 된다.
    - DB로 따지면 자식의 DAO나 repository가 없어도 된다.
- 도메인 주도 설계의 Aggregate Root 개념을 구현할 때 유용하다.
    - repository는 Aggregate Root만 컨택하고 나머지는 repository를 만들지 않는 방법
    - Aggregate Root를 통해서 생명주기를 관리한다.
        - Aggregate Root가 parent이고, child는 Aggreate Root가 관리한다.