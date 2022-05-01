# JPA에서 DTO 직접 조회

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-optimization/03/screenshot%202021-05-31%20오후%208.45.44.png)

- 화면이나 API에 fit한 용도로만 사용하는 쿼리는 패키지를 따로 파서 구현한다.
- 화면과 관련된 로직과 중요 핵심 비즈니스 로직을 분리할 수 있다.

{% tabs %} {% tab title="OrderQueryRepository.java" %}

```java

@Repository
@RequiredArgsConstructor
public class OrderQueryRepository {

    private final EntityManager em;

    public List<OrderQueryDto> findOrderQueryDtos() {
        List<OrderQueryDto> result = findOrders();

        result.forEach(o -> {
            List<OrderItemQueryDto> orderItems = findOrderItems(o.getOrderId());
            o.setOrderItems(orderItems);
        });
        return result;
    }

    // 컬렉션은 별도의 메서드로 조회한다.
    private List<OrderItemQueryDto> findOrderItems(Long orderId) {
        return em.createQuery(
                        "select new jpabook.jpashop.repository.order.query.OrderItemQueryDto(oi.order.id, i.name, oi.orderPrice, oi.count)" +
                                " from OrderItem oi" +
                                " join oi.item i" +
                                " where oi.order.id = : orderId", OrderItemQueryDto.class)
                .setParameter("orderId", orderId)
                .getResultList();
    }

    private List<OrderQueryDto> findOrders() {
        return em.createQuery(
                        "select new jpabook.jpashop.repository.order.query.OrderQueryDto(o.id, m.name, o.orderDate, o.status, d.address)" +
                                " from Order o" +
                                " join o.member m" +
                                " join o.delivery d", OrderQueryDto.class)
                .getResultList();
    }
}

```

{% endtab %} {% tab title="OrderApiController.java" %}

```java

@RestController
@RequiredArgsConstructor
public class OrderApiController {

    private final OrderRepository orderRepository;
    private final OrderQueryRepository orderQueryRepository;

    @GetMapping("/api/v4/orders")
    public List<OrderQueryDto> ordersV4() {
        return orderQueryRepository.findOrderQueryDtos();
    }
}
```

{% endtab %} {% tab title="OrderQueryDto.java" %}

```java

@Data
public class OrderQueryDto {
    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;
    private List<OrderItemQueryDto> orderItems;

    // 컬렉션은 생성자에서 제외한다.
    public OrderQueryDto(Long orderId, String name, LocalDateTime orderDate, OrderStatus orderStatus, Address address) {
        this.orderId = orderId;
        this.name = name;
        this.orderDate = orderDate;
        this.orderStatus = orderStatus;
        this.address = address;
    }
}
```

{% endtab %} {% tab title="OrderItemQueryDto.java" %}

```java

@Data
public class OrderItemQueryDto {
    private Long orderId;
    private String itemName;
    private int orderPrice;
    private int count;

    public OrderItemQueryDto(Long orderId, String itemName, int orderPrice, int count) {
        this.orderId = orderId;
        this.itemName = itemName;
        this.orderPrice = orderPrice;
        this.count = count;
    }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-optimization/03/screenshot%202021-05-31%20오후%209.11.37.png)

- 쿼리는 루트(order)에서 1번, 컬렉션(order_item)에서 N번 실행되었다.
- ToOne 관계를 먼저 조회하고 ToMany 관계는 별도로 처리한다.
    - ToOne은 join해도 데이터 row 수가 증가하지 않는다.
        - row가 증가하지 않기 때문에 join으로 최적화하기 쉬워 한 번에 조회한다.
    - ToMany는 join 하면 row 수가 증가한다.
        - 최적화하기 어려우므로 findOrderItems()같은 별도의 메서드로 조회한다.

## N+1

```java
public class OrderQueryRepository {

    private final EntityManager em;

    public List<OrderQueryDto> findOrderQueryDtos() {
        // query 1번 -> 결과는 N개
        List<OrderQueryDto> result = findOrders();

        // query N번
        result.forEach(o -> {
            List<OrderItemQueryDto> orderItems = findOrderItems(o.getOrderId());
            o.setOrderItems(orderItems);
        });
        return result;
    }
}
```

이 방식은 결과적으로 forEach를 돌면서 N+1을 초래한다.