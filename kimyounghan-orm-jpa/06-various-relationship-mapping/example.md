# 실전 예제

## Entity

![](../../.gitbook/assets/kimyounghan-orm-jpa/06/screenshot%202021-03-20%20오후%204.00.41.png)

order와 delivery는 1:1, product과 category는 N:M이다.

## ERD

![](../../.gitbook/assets/kimyounghan-orm-jpa/06/screenshot%202021-03-20%20오후%204.00.51.png)

member, order, delivery 관계에서 FK는 주 테이블인 orders에 넣는다. category와 item은 다대다 관계에서 중간 테이블을 둔다. 여기서는 연습용으로 @ManyToMany를 사용한다.

## Entity 상세

![](../../.gitbook/assets/kimyounghan-orm-jpa/06/screenshot%202021-03-20%20오후%204.00.57.png)

category와 item은 다대다 관계이지만 중간 테이블을 만들지 않았다.

## 연관 관계 구현

{% tabs %} {% tab title="Delivery.java" %}

```java

@Entity
public class Delivery {
    @Id
    @GeneratedValue
    private Long id;

    private String city;
    private String street;
    private String zipcode;

    private DeliveryStatus status;

    @OneToOne(mappedBy = "delivery")
    private Order order;
}
```

{% endtab %} {% tab title="Category.java" %}

```java

@Entity
public class Category {

    @Id
    @GeneratedValue
    private Long id;

    private String name;

    // 자기 자신을 매핑하는 것도 가능하다.
    @ManyToOne
    // 상위 카테고리가 연관 관계의 주인
    @JoinColumn(name = "PARENT_ID")
    private Category parent;

    // 각각의 상위 카테고리에 매핑 된 하위 카테고리
    @OneToMany(mappedBy = "parent")
    private List<Category> child = new ArrayList<>();

    @ManyToMany
    // 중간 테이블을 만들어준다.
    @JoinTable(
            // 중간 테이블 이름
            name = "CATEGORY_ITEM",
            // 한 쪽이 join 하는 것
            joinColumns = @JoinColumn(name = "CATEGORY_ID"),
            // 반대쪽이 join 하는 것
            inverseJoinColumns = @JoinColumn(name = "ITEM_ID")
    )
    private List<Item> items = new ArrayList<>();
}

```

{% endtab %} {% tab title="Item.java" %}

```java

@Entity
public class Item {

    @Id
    @GeneratedValue
    @Column(name = "item_id")
    private Long id;

    private String name;
    private int price;
    private int stockQuantity;

    // Category.items가 연관 관계의 주인이다.
    @ManyToMany(mappedBy = "items")
    private List<Category> categories = new ArrayList<>();
}
```

{% endtab %} {% tab title="Order.java" %}

```java

@Entity
@Table(name = "ORDERS")
public class Order {

    @Id
    @GeneratedValue
    @Column(name = "order_id")
    private Long id;

    @OneToOne
    @JoinColumn(name = "DELIVERY_ID")
    private Delivery delivery;
}
```

{% endtab %} {% endtabs %}

- 예제에서는 item과 category 사이에 `@ManyToMany`를 썼지만 실전에서는 사용하지 않는다.
    - 중간 테이블을 이용해 일대다, 다대일로 풀어낸다.
    - 실전에서는 중간 테이블이 단순하지 않기 때문이다.
    - 필드를 추가할 수도 없고 엔티티와 테이블이 일치하지 않아 제약이 많다.

### @JoinColumn

![](../../.gitbook/assets/kimyounghan-orm-jpa/06/screenshot%202021-03-20%20오후%204.01.11.png)

- 외래 키를 매핑할 때 사용한다.
- 외래 키가 참조하는 대상 테이블의 칼럼명이 다를 때는 referencedColumnName을 사용해 지정해준다.

### @ManyToOne

![](../../.gitbook/assets/kimyounghan-orm-jpa/06/screenshot%202021-03-20%20오후%204.01.16.png)

- fetch, cascade
    - 추후 설명한다.
- targetEntity
    - 제네릭이 없던 시절에 사용하던 옵션이라 몰라도 된다.

### @OneToMany

![](../../.gitbook/assets/kimyounghan-orm-jpa/06/screenshot%202021-03-20%20오후%204.01.22.png)

- 속성 설명에 다대일은 mappedBy가 없고 일대다는 있다.
    - 즉, 다대일이 붙은 곳은 꼭 연관 관계의 주인이 되어야 한다는 뜻이다.