# Entity를 DTO로 변환

```java

@RestController
@RequiredArgsConstructor
public class OrderApiController {
    @GetMapping("/api/v2/orders")
    public List<OrderDto> ordersV2() {
        List<Order> orders = orderRepository.findAll(new OrderSearch());

        return orders.stream().map(OrderDto::new).collect(Collectors.toList());
    }

    @Data
    static class OrderDto {
        private Long orderId;
        private String name;
        private LocalDateTime orderDate;
        private OrderStatus orderStatus;
        private Address address;
        private List<OrderItemDto> orderItems;

        public OrderDto(Order order) {
            orderId = order.getId();
            name = order.getMember().getName();
            orderDate = order.getOrderDate();
            orderStatus = order.getStatus();
            address = order.getDelivery().getAddress();
            orderItems = order.getOrderItems().stream().map(OrderItemDto::new)
                    .collect(Collectors.toList());
        }
    }

    @Data
    static class OrderItemDto {
        private String itemName;
        private int orderPrice;
        private int count;

        public OrderItemDto(OrderItem orderItem) {
            itemName = orderItem.getItem().getName();
            orderPrice = orderItem.getItem().getPrice();
            count = orderItem.getCount();
        }
    }
}
```

OrderDto에는 OrderItem이 아니라 OrderItemDto 형태로 있어야 한다. 안에 있는 필드도 Entity를 그대로 노출하면 안된다.

이 코드에서도 지연 로딩으로 인해 많은 SQL이 실행된다.

- order 1번
- member, address, orderItem N번
    - order 결과 개수만큼
- item N번
    - orderItem 결과 개수만큼

다만, 같은 Entity가 영속성 컨텍스트에 있다면 지연 로딩이더라도 SQL을 실행하지 않는다.

## 페치 조인 최적화

{% tabs %} {% tab title="findAllWithItem.java" %}

```java

@Repository
@RequiredArgsConstructor
public class OrderRepository {
    public List<Order> findAllWithItem() {
        return em.createQuery(
                "select o from Order o" +
                        " join fetch o.member m" +
                        " join fetch o.delivery d" +
                        " join fetch o.orderItems oi" +
                        " join fetch oi.item i", Order.class)
                .getResultList();
    }
}
```

{% endtab %} {% tab title="OrderApiController.java" %}

```java

@RestController
@RequiredArgsConstructor
public class OrderApiController {

    @GetMapping("/api/v3/orders")
    public List<OrderDto> ordersV3() {
        List<Order> orders = orderRepository.findAllWithItem();

        return orders.stream()
                .map(OrderDto::new)
                .collect(toList());
    }
}
```

{% endtab %} {% endtabs %}

fetch join을 썼지만 사실 한 가지 문제가 있다.

```sql
select *
from orders o
         join order_item oi on o.order_id = oi.order_id;
```

order와 order Item을 조인하면

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-optimization/03/screenshot%202021-05-30%20오후%207.53.46.png)

이처럼 중복된 결과가 나온다. order는 2개지만 order_item에는 각 order_id에 해당하는 데이터가 2개씩 있기 때문이다. 즉, N만큼 데이터가 뻥튀기 된다.

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-optimization/03/screenshot%202021-05-30%20오후%208.03.01.png)

게다가 뻥튀기 된 데이터는 레퍼런스마저 똑같다. JPA에서는 PK가 같으면 같은 참조값을 가지기 때문이다.

```java

@Repository
@RequiredArgsConstructor
public class OrderRepository {
    public List<Order> findAllWithItem() {
        return em.createQuery(
                "select distinct o from Order o" +
                        " join fetch o.member m" +
                        " join fetch o.delivery d" +
                        " join fetch o.orderItems oi" +
                        " join fetch oi.item i", Order.class)
                .getResultList();
    }
}
```

컬렉션의 데이터 뻥튀기를 막기 위해 distinct로 중복을 거른다.

## JPA의 distinct

- SQL에 distinct를 추가해서 실제 distinct 쿼리가 나간다.
    - DB의 distinct는 한 줄이 완전히 똑같아야 제거된다.
    - 하지만 몇몇 상황에서는 중복 데이터의 모든 칼럼 데이터가 똑같지 않아 제거되지 않는다.
- 조회 결과에 같은 Entity가 조회되면 애플리케이션에서 중복을 거른다.
    - 레퍼런스가 같은 중복 데이터를 날린다.
- 페이징이 불가능하다는 단점이 있다.
    - 컬렉션 fetch join에서 페이징을 사용하면 모든 데이터를 DB에서 일단 읽어온 뒤 메모리에서 페이징 하기 때문에 OOM이 발생할 수 있다.
  
## 컬렉션의 fetch join

- 컬렉션의 fetch join은 2개 이상의 컬렉션에 사용하면 안된다.
    - 1대 다의 다가 되면서 다 * 다가 되므로 데이터가 완전히 뻥튀기 되면서 부정합하게 조회된다.
