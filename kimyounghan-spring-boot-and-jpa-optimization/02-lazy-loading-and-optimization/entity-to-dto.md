# 엔티티를 DTO로 변환 

```java
@RestController
@RequiredArgsConstructor
public class OrderSimpleApiController {
    private final OrderRepository orderRepository;

    @GetMapping("/api/v2/simple-orders")
    public List<SimpleOrderDto> orderV2() {
        List<Order> orders = orderRepository.findAll(new OrderSearch());

        return orders.stream().map(SimpleOrderDto::new).collect(Collectors.toList());
    }

    @Data
    static class SimpleOrderDto {
        private Long orderId;
        private String name;
        private LocalDateTime orderDate;
        private OrderStatus orderStatus;
        private Address address;

        // DTO가 엔티티에 의존하는 것은 문제가 되지 않는다.
        public SimpleOrderDto(Order order) {
            orderId = order.getId();
            name = order.getMember().getName(); // lazy
            orderDate = order.getOrderDate();
            orderStatus = order.getStatus();
            address = order.getDelivery().getAddress(); // lazy
        }
    }
}

```

`name = order.getMember()`와 `order.getDelivery()`에서 연관 관계에 있는 데이터를 지연 로딩으로 불러온다.

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-optimization/02/screenshot%202021-05-30%20오후%206.08.42.png)

첫 번째 주문서는 order, order의 member, order의 delivery를 조회한다.

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-optimization/02/screenshot%202021-05-30%20오후%206.08.56.png)

두 번째 주문서도 member와 delivery를 다시 조회한다.

1. 맨 처음에 order를 조회하면서 SQL이 1번 실행된다.
2. 결과 데이터가 2개가 나온다.
3. map(SimpleOrderDto::new)에서 2번 루프를 돈다.
4. 처음 돌 때 해당 order에 대한 member와 delivery 쿼리를 날린다.
5. 두 번째 돌 때 해당 order에 대한 member와 delivery 쿼리를 날린다.

즉, order 조회 1번 + member 지연 로딩 N번 + delivery 지연 로딩 N번으로 1 + N + N번이 실행된다. order의 결과가 4개라면 최악의 경우 주문 1번 + 회원 4번 + 배송 4번이 실행된다.

최악이라고 한 이유는 같은 회원에 대한 정보를 조회한다면 영속성 컨텍스트에 존재하므로 쿼리가 나가지 않기 때문이다.

## fetch join 최적화

{% tabs %} {% tab title="OrderRepository.java" %}

```java
@Repository
@RequiredArgsConstructor
public class OrderRepository {
    public List<Order> findAllWithMemberDelivery() {
        return em.createQuery(
                "select o from Order o" + 
                        " join fetch o.member m" +
                        " join fetch o.delivery d", Order.class)
                .getResultList();
    }
}
```

{% endtab %} {% tab title="OrderSimpleApiController.java" %}

```java
@RestController
@RequiredArgsConstructor
public class OrderSimpleApiController {
    private final OrderRepository orderRepository;

    @GetMapping("/api/v3/simple-orders")
    public List<SimpleOrderDto> ordersV3() {
        List<Order> orders = orderRepository.findAllWithMemberDelivery();

        return orders.stream().map(SimpleOrderDto::new)
                .collect(Collectors.toList());
    }
}
```

{% endtab %} {% endtabs %}

![](../../.gitbook/assets/kimyounghan-spring-boot-and-jpa-optimization/02/screenshot%202021-05-30%20오후%206.33.59.png)

쿼리 한 방으로 불러와졌다. order와 member, order와 delivery를 한 번에 조인해서 가져온다.

- 엔티티를 fetch join을 사용해 쿼리 1번으로 조회한다.
- 연관 관계에 있는 값을 프록시 대신 실제 값으로 다 채워서 가져온다.
- fetch join으로 member, delivery는 이미 조회된 상태이므로 지연 로딩 하지 않는다.

실무에서 fetch join을 적극적으로 사용하는 것이 좋다.