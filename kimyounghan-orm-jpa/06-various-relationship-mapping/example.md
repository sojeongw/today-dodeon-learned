# 실전 예제
## Entity

![](../../.gitbook/assets/kimyounghan-orm-jpa/06/screenshot%202021-03-20%20오후%204.00.41.png)

주문과 배송은 1:1, 상품과 카테고리는 N:M으로 되어있다.

## ERD

![](../../.gitbook/assets/kimyounghan-orm-jpa/06/screenshot%202021-03-20%20오후%204.00.51.png)

회원, 주문, 배송 관계에서 FK는 주 테이블인 `ORDERS`에 넣기로 결정했다. 카테고리와 아이템은 다대다 관계에서 중간 테이블을 둔다. 여기서는 연습용으로 `@ManyToMany`를 사용한다.

## Entity 상세

![](../../.gitbook/assets/kimyounghan-orm-jpa/06/screenshot%202021-03-20%20오후%204.00.57.png)

카테고리와 아이템을 보면 테이블과 달리 중간 Entity는 없다.

## 연관 관계 구현

{% tabs %} {% tab title="Delivery.java" %}

```java
@Entity
public class Delivery {
  @Id @GeneratedValue
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

  @ManyToOne
  @JoinColumn(name = "PARENT_ID")
  private Category parent;  // 상위 카테고리

  @OneToMany(mappedBy = "parent")
  private List<Category> child = new ArrayList<>(); // 하위 카테고리

  @ManyToMany
  // 중간 테이블을 만들어준다.
  @JoinTable(
      // 중간 테이블 이름
      name = "CATEGORY_ITEM",
      // 내가 join 하는 것
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

예제에서는 `@ManyToMany`를 썼지만 실전에서는 사용하지 않는다. 중간 테이블을 이용해 일대다, 다대일로 풀어낸다. 실전에서는 중간 테이블이 단순하지 않기 때문이다. `@ManyToMany`는 필드를 추가할 수도 없고 Entity 테이블이 일치하지 않아 제약이 많다.

### @JoinColumn

![](../../.gitbook/assets/kimyounghan-orm-jpa/06/screenshot%202021-03-20%20오후%204.01.11.png)

외래 키를 매핑할 때 사용한다. 가끔가다 외래 키가 참조하는 대상 테이블의 칼럼명이 다를 때가 있다. 이때는 `referencedColumnName`을 사용해 지정해준다.

### @ManyToOne

![](../../.gitbook/assets/kimyounghan-orm-jpa/06/screenshot%202021-03-20%20오후%204.01.16.png)

`fetch`나 `cascade`는 뒤에서 설명할 예정이다. `targetEntity`는 몰라도 된다. 제네릭이 없던 시절에 사용하던 옵션이다.

### @OneToMany

![](../../.gitbook/assets/kimyounghan-orm-jpa/06/screenshot%202021-03-20%20오후%204.01.22.png)

다대일과 달리 일대다는 `mappedBy`가 있다. 이말은 곧 다대일을 쓰면 꼭 연관 관계의 주인이 되어야 한다는 것이다.