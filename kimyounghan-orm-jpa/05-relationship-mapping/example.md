# 실전 예제

![](../../.gitbook/assets/kimyounghan-orm-jpa/05/screenshot%202021-03-20%20오전%2012.27.33.png)

테이블 구조는 이전과 같다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/05/screenshot%202021-03-20%20오전%2012.27.38.png)

객체 구조는 참조를 사용하도록 변경되었다. `member`는 `orders`를, `order`는 `member`와 `orderItems`, `orderItem`은 `item`
과 `order`를 참조한다. 대부분의 연관 관계를 세팅했다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/05/screenshot%202021-03-20%20오전%2012.27.33.png)

앞서 설명했듯 단방향 연관 관계 매핑과 연관 관계 주인을 설정하는 것이 중요하다. 테이블 구조를 다시 보고 외래키만 잘 넣어주면 된다.

{% endtab %} {% tab title="before" %}

```java

@Entity
@Table(name = "ORDERS")
public class Order {

  @Id
  @GeneratedValue
  @Column(name = "order_id")
  private Long id;

  // 누가 주문했는지 알기 위한 용도
  @Column(name = "member_id")
  private Long memberId;

  private LocalDateTime orderDate;

  @Enumerated(EnumType.STRING)
  private OrderStatus status;
}

```

{% endtab %} {% tab title="after" %}

```java

@Entity
@Table(name = "ORDERS")
public class Order {

  @Id
  @GeneratedValue
  @Column(name = "order_id")
  private Long id;

  // 회원에게 주문은 여러 개지만, 주문을 갖고있는 회원은 한 명이다.
  @ManyToOne
  @JoinColumn(name = "MEMBER_ID")
  private Member member;

  private LocalDateTime orderDate;

  @Enumerated(EnumType.STRING)
  private OrderStatus status;
}

```

{% endtab %} {% endtabs %}

{% endtab %} {% tab title="before" %}

```java

@Entity
public class OrderItem {

  @Id
  @GeneratedValue
  @Column(name = "order_item_id")
  private Long id;

  @Column(name = "order_id")
  private Long orderId;

  @Column(name = "item_id")
  private Long itemId;

  @Enumerated(EnumType.STRING)
  private OrderStatus status;
}
```

{% endtab %} {% tab title="after" %}

```java

@Entity
public class OrderItem {

  @Id
  @GeneratedValue
  @Column(name = "order_item_id")
  private Long id;

  @ManyToOne
  // 여기서 외래키를 관리하니까 연관 관계의 주인이 된다.
  @JoinColumn(name = "ORDER_ID")
  private Order order;

  @ManyToOne
  @JoinColumn(name = "ITEM_ID")
  private Item item;

  @Enumerated(EnumType.STRING)
  private OrderStatus status;
}

```

{% endtab %} {% endtabs %}

일단 단방향으로 설계를 완료한다. 그 후에 양방향이 필요한 곳이 있는지 확인해보자.

![](../../.gitbook/assets/kimyounghan-orm-jpa/05/screenshot%202021-03-20%20오전%2012.27.33.png)

보통 회원이 주문을 가지고 있는 건 좋은 설계가 아니다. 특정 회원의 주문을 보고 싶을 때 DB 기준으로 보면 `order`에 `member_id`가 외래키로 있기 때문에 이걸로도
충분히 조회할 수 있다.

회원을 찾아서 그 안에서 `getOrders()`를 해서 주문을 뿌리는 것은 설계 미스다. 관심사를 제대로 못 끊어낸 것이다. 이런 걸 잘 끊어내는 것이 설계에서 중요하다. 주문이
필요하면 주문에서 시작하면 되는 것이다.

하지만 굳이 양방향 관계가 필요하다고 치고 넣는다면 아래와 같다.

{% endtab %} {% tab title="before" %}

```java

@Entity
public class Member {

  @Id
  @GeneratedValue(strategy = GenerationType.AUTO)
  @Column(name = "member_id")
  private Long id;
  private String name;
  private String city;
  private String street;
  private String zipcode;
}

```

{% endtab %} {% tab title="after" %}

```java

@Entity
public class Member {

  @Id
  @GeneratedValue(strategy = GenerationType.AUTO)
  @Column(name = "member_id")
  private Long id;
  private String name;
  private String city;
  private String street;
  private String zipcode;

  // 굳이 회원에서 PK를 관리해서 양방향이 필요하다면 추가한다.
  // 연관 관계의 주인은 order에 있는 member가 된다.
  @OneToMany(mappedBy = "member")
  private List<Order> orders = new ArrayList<>();
}

```

{% endtab %} {% endtabs %}

{% endtab %} {% tab title="before" %}

```java

@Entity
@Table(name = "ORDERS")
public class Order {

  @Id
  @GeneratedValue
  @Column(name = "order_id")
  private Long id;

  @Column(name = "member_id")
  private Long memberId;

  private LocalDateTime orderDate;

  @Enumerated(EnumType.STRING)
  private OrderStatus status;
}

```

{% endtab %} {% tab title="after" %}

```java

@Entity
@Table(name = "ORDERS")
public class Order {

  @Id
  @GeneratedValue
  @Column(name = "order_id")
  private Long id;

  @ManyToOne
  @JoinColumn(name = "MEMBER_ID")
  private Member member;

  // 주문에서 주문과 연관된 상품을 불러오는 것은 매우 자주 있는 일이다.
  // 주문서를 중심으로 불러올 수 있도록 양방향 연관 관계를 설정한다.
  // orderItem에 있는 order가 연관 관계의 주인이다.
  @OneToMany(mappedBy = "order")
  private List<OrderItem> orderItems = new ArrayList<>();

  private LocalDateTime orderDate;

  @Enumerated(EnumType.STRING)
  private OrderStatus status;
}

```

{% endtab %} {% endtabs %}

이제 주문을 해보자.

{% endtab %} {% tab title="JpaMain.java" %}

```java
public class JpaMain {

  public static void main(String[] args) {
    try {
      Order order = new Order();

      // 연관 관계 편의 메서드
      order.addOrderItem(new OrderItem());

      em.persist(orderItem);
      tx.commit();
    } catch (Exception e) {
      tx.rollback();
    } finally {
      em.close();
    }
    entityManagerFactory.close();
  }
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

  @ManyToOne
  @JoinColumn(name = "MEMBER_ID")
  private Member member;

  // 주문에서 주문과 연관된 상품을 불러오는 것은 매우 자주 있는 일이다.
  // 주문서를 중심으로 불러올 수 있도록 양방향 연관 관계를 설정한다.
  // orderItem에 있는 order가 연관 관계의 주인이다.
  @OneToMany(mappedBy = "order")
  private List<OrderItem> orderItems = new ArrayList<>();

  private LocalDateTime orderDate;

  @Enumerated(EnumType.STRING)
  private OrderStatus status;

  // 연관 관계 편의 메서드
  public void addOrderItem(OrderItem orderItem) {
    orderItems.add(orderItem);
    orderItem.setOrder(this);
  }
}
```

{% endtab %} {% endtabs %}

연관 관계 메서드를 통해 양방향 매핑을 할 수 있다.

```java
public class JpaMain {

  public static void main(String[] args) {
    try {
      Order order = new Order();
      em.persist(order);

      // 양방향이 아니어도 이렇게 진행할 수 있다.
      OrderItem orderItem = new OrderItem();
      orderItem.setOrder(order);
      em.persist(orderItem);

      tx.commit();
    } catch (Exception e) {
      tx.rollback();
    } finally {
      em.close();
    }

    entityManagerFactory.close();
  }
}
```

이때 이렇게 `orderItem`에서 `order`를 추가해줄 수도 있다. 이렇게 양방향 매핑관계가 아니어도 얼마든지 문제를 해결할 수 있다.
