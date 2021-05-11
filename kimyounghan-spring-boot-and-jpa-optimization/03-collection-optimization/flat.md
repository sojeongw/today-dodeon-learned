# 플랫 데이터 최적화

{% tabs %} {% tab title="OrderApiController.java" %}

```java

@RestController
@RequiredArgsConstructor
public class OrderApiController {

    @GetMapping("/api/v6/orders")
    public List<OrderFlatDto> ordersV6() {
        List<OrderFlatDto> flats = orderQueryRepository.findAllByDto_flat();

        return flats;
    }
}
```

{% endtab %} {% tab title="OrderQueryRepository.java" %}

```java

@Repository
@RequiredArgsConstructor
public class OrderQueryRepository {

    public List<OrderFlatDto> findAllByDto_flat() {
        // 모든 데이터를 join 한다.
        return em.createQuery("select new jpabook.jpashop.repository.order.query.OrderFlatDto(o.id, m.name, o.orderDate, o.status, d.address, i.name, oi.orderPrice, oi.count)" +
                " from Order o" +
                " join o.member m" +
                " join o.delivery d" +
                " join o.orderItems oi" +
                " join oi.item i", OrderFlatDto.class)
                .getResultList();
    }
}
```

{% endtab %} {% tab title="OrderFlatDto.java" %}

```java

@Data
public class OrderFlatDto {

    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private Address address;
    private OrderStatus orderStatus;

    private String itemName;
    private int orderPrice;
    private int count;

    public OrderFlatDto(Long orderId, String name, LocalDateTime orderDate, OrderStatus orderStatus, Address address, String itemName, int orderPrice, int count) {
        this.orderId = orderId;
        this.name = name;
        this.orderDate = orderDate;
        this.address = address;
        this.orderStatus = orderStatus;
        this.itemName = itemName;
        this.orderPrice = orderPrice;
        this.count = count;
    }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-optimization/03/screenshot%202021-05-31%20오후%209.47.17.png)

모든 데이터를 join 해서 조회하면 중복 데이터가 반환된다. order가 아니라 orderItem이 기준이 되면서 페이징이 불가하다.

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-optimization/03/screenshot%202021-05-31%20오후%209.49.43.png)

하지만 쿼리가 총 한 번만 나간다는 장점이 있다.

{% tabs %} {% tab title="OrderApiController.java" %}

```java

@RestController
@RequiredArgsConstructor
public class OrderApiController {

    @GetMapping("/api/v6/orders")
    public List<OrderQueryDto> ordersV6() {
        List<OrderFlatDto> flats = orderQueryRepository.findAllByDto_flat();

        return flats.stream()
                .collect(groupingBy(o -> new OrderQueryDto(o.getOrderId(), o.getName(), o.getOrderDate(), o.getOrderStatus(), o.getAddress()),
                        mapping(o -> new OrderItemQueryDto(o.getOrderId(), o.getItemName(), o.getOrderPrice(), o.getCount()), toList()))).entrySet().stream()
                .map(e -> new OrderQueryDto(e.getKey().getOrderId(), e.getKey().getName(), e.getKey().getOrderDate(),
                        e.getKey().getOrderStatus(), e.getKey().getAddress(), e.getValue()))
                .collect(toList());
    }
}
```

{% endtab %} {% tab title="OrderQueryDto.java" %}

```java

@Data
// 컨트롤러에서 groupingBy할 때 orderId 기준으로 묶어서 중복을 없애준다.
@EqualsAndHashCode(of = "orderId")
public class OrderQueryDto {
    private Long orderId;
    private String name;
    private LocalDateTime orderDate;
    private OrderStatus orderStatus;
    private Address address;
    private List<OrderItemQueryDto> orderItems;

    public OrderQueryDto(Long orderId, String name, LocalDateTime orderDate, OrderStatus orderStatus, Address address) {
        this.orderId = orderId;
        this.name = name;
        this.orderDate = orderDate;
        this.orderStatus = orderStatus;
        this.address = address;
    }

    public OrderQueryDto(Long orderId, String name, LocalDateTime orderDate, OrderStatus orderStatus, Address address, List<OrderItemQueryDto> orderItems) {
        this.orderId = orderId;
        this.name = name;
        this.orderDate = orderDate;
        this.orderStatus = orderStatus;
        this.address = address;
        this.orderItems = orderItems;
    }
}
```

{% endtab %} {% endtabs %}

API 스펙을 OrderQueryDto로 맞춰야 한다면 이런 식으로 변환할 수 있다.

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-optimization/03/screenshot%202021-05-31%20오후%2010.00.05.png)

`@EqualsAndHashCode(of = "orderId")` 덕분에 중복도 제거되었다.

## 단점

- 쿼리는 한 번이지만 join으로 인해 DB에서 애플리케이션에 전달하는 데이터에 중복이 추가된다.
    - 상황에 따라 V5보다 더 느릴 수도 있다.
- 애플리케이션에서 해야 할 추가 작업이 크다.
    - 분해하는 추가 작업이 필요하다.
- 페이징이 불가능하다.
    - 데이터가 중복되기 때문에 정확한 페이징 결과가 나오지 않는다.
    - 2개만 페이지 하면 중복 데이터 2개만 나오게 된다.