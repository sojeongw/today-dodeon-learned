# 실전 예제

![](../../.gitbook/assets/kimyounghan-orm-jpa/05/screenshot%202021-03-20%20오전%2012.27.33.png)

테이블 구조는 이전과 같다. 아이템엔 FK가 없다. 주문에서 어떤 아이템이 있는지는 조회해도 아이템 당 어떤 주문인지는 잘 파악하지 않기 떄문이다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/05/screenshot%202021-03-20%20오전%2012.27.38.png)

객체 구조는 참조를 사용하도록 변경되었다.

![](../../.gitbook/assets/kimyounghan-orm-jpa/05/screenshot%202021-03-20%20오전%2012.27.33.png)

단방향 연관 관계 매핑과 연관 관계 주인을 설정하는 것이 중요하다. 테이블 구조를 다시 보고 외래키만 잘 넣어주면 된다.

## 단방향 연관관계 설정

{% tabs %} {% tab title="before" %}

```java

@Entity
@Table(name = "ORDERS")
public class Order {

    @Id
    @GeneratedValue
    @Column(name = "order_id")
    private Long id;

    // FK만 들어가있다.
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

    // Member 객체 자체를 참조하고 FK를 기준으로 join 한다.
    @ManyToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;

    private LocalDateTime orderDate;

    @Enumerated(EnumType.STRING)
    private OrderStatus status;
}

```

{% endtab %} {% endtabs %}

- 테이블 중심의 설계에서 객체를 참조하는 형태로 변경했다.

{% tabs %} {% tab title="before" %}

```java

@Entity
public class OrderItem {

    @Id
    @GeneratedValue
    @Column(name = "order_item_id")
    private Long id;

    // order와 item 모두 FK만 존재한다.
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

    // 외래키를 관리하니까 OrderItem.order와 OrderItem.item이 연관 관계의 주인이 된다.
    @ManyToOne
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

### 주문 조회 설계

![](../../.gitbook/assets/kimyounghan-orm-jpa/05/screenshot%202021-03-20%20오전%2012.27.33.png)

- 특정 회원의 주문을 보고 싶다고 회원이 주문을 가지고 있는 건 좋은 설계가 아니다.
    - member를 찾아서 `getOrders()` 하는 단계를 거치는 건 설계 미스다.
    - 주문이 필요하면 주문에서 시작해야 한다. 회원에서 주문을 찾는 건 관심사를 제대로 못 끊어낸 것이다.
- 어차피 DB 다이어그램을 보면 order에 member_id가 외래 키로 있어 조회 가능하다.

## 양방향 연관 관계 설정

{% tabs %} {% tab title="Member.java" %}

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

    // 연관 관계의 주인은 Order.member가 된다.
    @OneToMany(mappedBy = "member")
    private List<Order> orders = new ArrayList<>();
}

```

{% endtab %} {% endtabs %}

필요하진 않지만 연습을 위해 적용해본다.

{% tabs %} {% tab title="Order.java" %}

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

    // 비즈니스적으로 가치가 있는 값은 양방향으로 설정한다.
    // OrderItem.order가 연관 관계의 주인이다.
    @OneToMany(mappedBy = "order")
    private List<OrderItem> orderItems = new ArrayList<>();

    private LocalDateTime orderDate;

    @Enumerated(EnumType.STRING)
    private OrderStatus status;
}

```

{% endtab %} {% endtabs %}

주문은 주문과 연관된 아이템 목록을 가져오는 기능이 필요하므로 양방향을 설정해준다.

{% tabs %} {% tab title="Order.java" %}

```java

@Entity
@Table(name = "ORDERS")
public class Order {

    ...

    // Order와 OrderItem이 양방향 연관 관계이므로 연관 관계 편의 메서드를 구현한다.
    public void addOrderItem(OrderItem orderItem) {
        orderItems.add(orderItem);
        orderItem.setOrder(this);
    }
}
```

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

{% endtab %} {% endtabs %}

연관 관계 메서드를 통해 양 쪽 모두 값을 세팅한다.

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

물론 단방향 그대로 유지해도 OrderItem을 가져와서 세팅해주면 된다. 양방향 설정이 굳이 필요없다. 즉, Order에 OrderItem 참조를 추가하지 않아도 된다.

최대한 단방향으로 하고 실무에서 JPQL을 사용할 때나 비즈니스 상 양쪽으로 참조가 있어야 순조로울 때가 오면 양방향으로 설정하자. 
